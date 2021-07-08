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
    'montana.cpp',
  ],
  headers = [
    'montana.h',
  ],
  deps = [
    ':monty',
  ],
)

cxx_library(
  name = 'monty',
  srcs = [
    'monty.cpp',
  ],
  headers = [
    'monty.h',
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

So first, let's pick a `dist` we want to use, in this case I'll be using `focal`, I'll be setting my `language` to generic, and you'll see something rather specific, which is we will not be using `Homebrew` in this `.travis.yml` file, but `Linuxbrew`, let's get started: 

```yaml

dist: focal
language: generic

services: docker 

before_install:
  # Install Linuxbrew
  - test -d $HOME/.linuxbrew/bin || git clone https://github.com/Linuxbrew/brew.git $HOME/.linuxbrew
  - PATH="$HOME/.linuxbrew/bin:$PATH"
  - echo 'export PATH="$HOME/.linuxbrew/bin:$PATH"' >>~/.bash_profile
  - export MANPATH="$(brew --prefix)/share/man:$MANPATH"
  - export INFOPATH="$(brew --prefix)/share/info:$INFOPATH"

  # Install Buck
  - brew tap facebook/fb
  - brew install buck
  - buck --version
  # Install GCC
  - brew install gcc
  
```
Now at this point, you've picked your `language`, `dist`, and now you've fetched `Linuxbrew`. Let's now use the `script` hook in Travis to utilize Buck:

```yaml
script:
  - 'buck build :montana'
  directories:
    - $HOME/.linuxbrew/
 ```
 As you can see Travis is going to run `buck build` and we also are caching the `linuxbrew` directory. It is all about speed right? So in it's entirety, your `.travis.yml` file should look like this: 
 
 ```yaml
 dist: focal
language: generic
services: docker
before_install:
  - test -d $HOME/.linuxbrew/bin || git clone https://github.com/Linuxbrew/brew.git $HOME/.linuxbrew
  - 'PATH="$HOME/.linuxbrew/bin:$PATH"'
  - 'echo ''export PATH="$HOME/.linuxbrew/bin:$PATH"'' >>~/.bash_profile'
  - 'export MANPATH="$(brew --prefix)/share/man:$MANPATH"'
  - 'export INFOPATH="$(brew --prefix)/share/info:$INFOPATH"'
  - brew --version
  - brew tap facebook/fb
  - brew install buck
  - buck --version
  - brew install gcc
  - echo buck testing! 
script:
  - 'buck build :montana'
  directories:
    - $HOME/.linuxbrew/
  ```

After all said and done and you trigger the build, you should see similar results to this: 

<img width="805" alt="Screen Shot 2021-07-07 at 2 14 52 PM" src="https://user-images.githubusercontent.com/20936398/124829810-f57c0480-df2d-11eb-8b87-69ceb5773169.png">

As you can see my build with Buck was successful with Travis, and there you have it! If you have any questions please feel free to contact me at [montana@travis-ci.com](mailto:montana@travis-ci.com), and happy building! 
