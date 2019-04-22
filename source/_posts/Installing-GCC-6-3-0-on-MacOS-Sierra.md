---
title: Installing GCC 6.3.0 on MacOS Sierra
date: 2016-12-23 15:10:48
---

GCC 6.3 was released on December 21, 2016. It's unlikely that most platforms will offer updates  by the end of the year. 

Compiling from source is the the fastest and surest way to use latest and greatest features from GCC 6.3. 

I will provide the steps needed to compile everything from source. 

### Step One

You need a compiler to compile a compiler.

Any ISO C++98 compilier will do. Please check if you have gcc already installed by running on Terminal `gcc --version`. Otherwise, installing xcode command line tools  will do the trick:

```bash
xcode-select --install
```

### Step Two

GCC requires a few support libraries to be compiled.

The most basic ones to a minimal working compiler are:

* GNU Multiple Precision Library (GMP) version 4.3.2 (or later)
* MPFR Library version 2.4.2 (or later)
* MPC Library version 0.8.1 (or later)

Create a folder and download the latest version of the above libraries : [GMP](https://gmplib.org/download/gmp/gmp-6.1.2.tar.bz2) , [MPFR](http://www.mpfr.org/mpfr-current/mpfr-3.1.5.tar.bz2) and  [MPC](ftp://ftp.gnu.org/gnu/mpc/mpc-1.0.3.tar.gz) . 

### Step Three

Compiling support libraries:

#### GMP

Extract the compressed GMP package to a folder `gmp-6.1.2`.

cd into the `gmp-6.1.2`. and make a new folder called `build`:

```bash
cd gmp-6.1.2
mkdir build
```

I recommend installing the user compiled compilers within the folder `/usr/local/`. For GCC 6.3, the recommended folder  is `/usr/local/gcc-6.3.0`

Make sure that your set-up has all the requirements needed to compile GMP by running:

```
../configure --prefix=/usr/local/gcc-6.3.0 --enable-cxx
```

If no errors are reported, we can go ahead and build GMP

```
make -j 4
```

The `-j 4` parameters tell make to run 4 jobs in parallel. I chose 4 since I have a quad-core processor.

Install GMP into  the folder `/usr/local/gcc-6.3.0` by:

```
sudo make install
```

#### MPFR

Using the same steps as for GMP:

Extract and create a build folder:

```
cd mpfr-3.1.5
mkdir build
```

Configure:

```
../configure --prefix=/usr/local/gcc-6.3.0 --with-gmp=/usr/local/gcc-6.3.0
```

Build:

```
make -j 4
```

Install:

```
sudo make install
```

#### MPC

Using the same steps as above:

Extract and create a build folder:

```
cd mpc-1.0.3
mkdir build
```

Configure:

```
../configure --prefix=/usr/local/gcc-6.3.0 --with-gmp=/usr/local/gcc-6.3.0 --with-mpfr=/usr/local/gcc-6.3.0
```

Build:

```
make -j 4
```

Install:

```
sudo make install
```

#### Finally GCC

Go ahead and [download GCC 6.3](https://gcc.gnu.org/mirrors.html) if you haven't already. Most of these mirrors are ftp servers. You should navigate to `/gcc/releases/gcc-6.3.0/` and download a bz2 or gz package. 

Using the same steps as above:

Extract and create a build folder:

```
cd gcc-6.3.0
mkdir build
```

Configure:

```
../configure --prefix=/usr/local/gcc-6.3.0 --enable-checking=release --with-gmp=/usr/local/gcc-6.3.0 --with-mpfr=/usr/local/gcc-6.3.0 --with-mpc=/usr/local/gcc-6.3.0 --enable-languages=c,c++,fortran --with-isl=/usr/local/gcc-6.3.0 --program-suffix=-6.3.0
```

Build:

```
make -j 4
```

NOTE: This process will take some time. On my machine it took about one hour ten minutes. The resulting build will be about 4 GB.

Install:

```
sudo make install
```

Add the `/usr/local/gcc-6.3.0` folder to PATH by adding the following line to your `.bashrc`  file if you use Bash or `.zshrc` if you use ZSH  and so on.

```
export PATH=/usr/local/gcc-6.3.0/bin:$PATH
```

Close and open you Terminal application.

At this point you are done.

Test the installation by running:

```bash
g++-6.3.0 --version
```

You should get something similar to this:

```
g++-6.3.0 (GCC) 6.3.0
Copyright (C) 2016 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
```

NOTE: I like having many compiler versions in my system. I do not want to interfere with the default gcc or g++ that comes with Xcode. But you can go ahead and sym link `g++-6.3.0` to `g++`. 

### Testing

Let's test the compiler using the [fold expression](http://en.cppreference.com/w/cpp/language/fold) feature in C++17. Fold expressions are like `reduce` in Python and introduce a whole new level of magic in C++:

```cpp
#include <iostream>

/**
Computes the product of numbers using fold expressions
**/

template<typename... T>
auto fold_product(T... s){
	return (... * s);
}

int main(int argc, char const *argv[])
{
	std::cout << "Product using folds: " << fold_product(1,2,3,4) << std::endl;
	return 0;
}
```

Compile with:

```
g++-6.3.0 test.cpp -o test --std=c++1z
```

Running `./test` you should see:

```
Product using folds: 24
```

Now that you have all this power in GCC 6.3, go make something. 

Thanks for your time.  If you have any suggestions, corrections and comments, feel free to reach out to me. 