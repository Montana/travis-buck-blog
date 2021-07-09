---
title: "Travis CI + Buck"
created_at: Fri Jul 09 2021 15:00:00 EDT
author: Montana Mendy
layout: post
permalink: 2021-07-09-buck
category: news
excerpt_separator: <!-- more --> 
tags:
  - news
  - feature
  - infrastructure
  - community
---


![TCI-Graphics for AdsBlogs](https://user-images.githubusercontent.com/20936398/124826241-7684cd00-df29-11eb-8fcb-21db161a5087.png)


So you have a Buck based project and want to use Travis as your CI, first that's a great choice, secondly Buck is quick. Buck is a build system developed and used by Facebook. It encourages the creation of small, reusable modules consisting of code and resources, and supports a variety of languages on many platforms. In this weeks tutorial I'll be showing you how to integrate Buck into your Travis CI builds.

<!-- more --> 

## Buck up and let's get building


A few things you'll want to have, first thing is a file entitled `BUCK`, I wrote up a little example of how this file should look like:

```cpp
cxx_binary(
  name = 'montana',
  srcs = [
    'main.cpp',
  ],
  compiler_flags = [
    '-v',
  ],
)
```

Now let's edit our file entitled `montana.cpp`, this is a quick C++ program I've made that will reverse sentences using recursion: 

```cpp
#include <iostream>
using namespace std;

void rev_str(char* string)

{
  if (*string == '\0')

    return;

  else

  {
    rev_str(string + 1);

    cout << *string;
  }
}

int main()

{
  char string[] = "Building with Travis CI";

  cout << "Original String: " << string << endl;

  cout << "Reversed String: ";

  rev_str(string);

  return 0;
}
```

Alright we now have our `BUCK` file, and our C++ program! Let's start setting up Travis for Buck. 

## The setup 

So first, let's pick a `dist` we want to use, in this case I'll be using `trusty`, I'll be setting my `language` to generic, and you'll see something rather specific, which is we will not be using `Homebrew` in this `.travis.yml` file, but `Linuxbrew`, let's get started: 

```yaml
language: cpp
sudo: true
dist: trusty

addons:
  apt:
    sources:
      - ubuntu-toolchain-r-test
    packages:
      - g++-6
      - gcc-6

before_install:
- sudo update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-6 90
- sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-6 90
- sudo apt-get install -y equivs openjdk-8-jdk
- wget -O buck.deb https://github.com/facebook/buck/releases/download/v2018.08.27.01/buck.2018.08.27.01_all.deb
- sudo dpkg -i buck.deb
- buck --version
- wget -O buckaroo.deb https://github.com/LoopPerfect/buckaroo/releases/download/v1.4.1/buckaroo.deb
- sudo dpkg -i buckaroo.deb
- buckaroo version
- c++ --version
- g++ --version
- gcc --version

script:
- buckaroo install
- buck build :montana
- buck test :montana
```

Now at this point, you've picked your `language`, `dist`, and now you've fetched `Linuxbrew`. Let's now use the `script` hook in Travis to utilize Buck:

```yaml
script:
- buckaroo install
- buck build :montana
- buck test :montana
  ```
 As you can see Travis is going to run `buck build` and we also are caching the `linuxbrew` directory. It is all about speed right? So in it's entirety, your `.travis.yml` file should look like this: 
 
```yaml
language: cpp
sudo: true
dist: trusty

addons:
  apt:
    sources:
      - ubuntu-toolchain-r-test
    packages:
      - g++-6
      - gcc-6

before_install:
- sudo update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-6 90
- sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-6 90
- sudo apt-get install -y equivs openjdk-8-jdk
- wget -O buck.deb https://github.com/facebook/buck/releases/download/v2018.08.27.01/buck.2018.08.27.01_all.deb
- sudo dpkg -i buck.deb
- buck --version
- wget -O buckaroo.deb https://github.com/LoopPerfect/buckaroo/releases/download/v1.4.1/buckaroo.deb
- sudo dpkg -i buckaroo.deb
- buckaroo version
- c++ --version
- g++ --version
- gcc --version
```

After all said and done and you trigger the build, you should see similar results to this: 

<img width="805" alt="Screen Shot 2021-07-07 at 2 14 52 PM" src="https://user-images.githubusercontent.com/20936398/124829810-f57c0480-df2d-11eb-8b87-69ceb5773169.png">


## Buckaroo 

You'll also not want to forget to create a file called `Buckaroo.json`, this file should look a bit something like this: 

```json

{
  "name": "montana",
  "dependencies": {
  }
}
```

You also need a `.buckconfig` file, you'll notice we instructed Travis to grab Buckaroo via these lines:

```yaml
- wget -O buckaroo.deb https://github.com/LoopPerfect/buckaroo/releases/download/v1.4.1/buckaroo.deb
- sudo dpkg -i buckaroo.deb
- buckaroo version
```
This is use `wget` to fetch `Buckaroo`, and then printing the version out for a more verbose look at your Travis CI log. Let's move onto the `.buckconfig` file. 

# Buck config 

Let's create a file called `.buckconfig`, this is an exact copy of what mine looks like currently, feel free to use it:

```starlark

[project]
  ignore = .git, .buckd

[parser]
  default_build_file_syntax = SKYLARK

[cxx]
  should_remap_host_platform = true

[cxx#linux-x86_64]
  cxxflags = -std=c++14

[cxx#macosx-x86_64]
  cxxflags = -std=c++14
```

When it comes to package managers you fetch from, you'll want to use `cURL`, on that note you have a choice obviously of using `Linuxbrew` or `Homebrew`. In my example I used `Linuxbrew`, this is just a choice, you can use `Homebrew` if you wish, you need to make sure you have something called a `Brewfile`, create a file called `Brewfile`, and add the following: 

```bash
tap 'facebook/fb'
brew 'buck'
brew 'xctool'
```

As you can see my build with Buck was successful with Travis, and there you have it! If you have any questions please feel free to contact me at [montana@travis-ci.com](mailto:montana@travis-ci.com), and happy building! 
