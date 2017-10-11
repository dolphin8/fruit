# fruit
deploy c++ binary easily (once build, run everywhere)

# Background
Java have the ability to be compiled once and run everywhere, and sometimes we hope that can be true to C++. Consider when you have to work in an very old environment with very old version of gcc & glibc, and you have no permission to make changes to the system. Since this year is 2017, we certainly want to write cpp code in C++11, C++14, C++17. Unfortunally, all the compatiable compilers are not avaliable. To compile a whole toolchain takes a lot of time, and sadly, you have to walk through tons of failures, becasue the old environment lacks some dependence. 

I have tried Junest, but because lack of root permission, I can only use it the fakeroot way. I have also tried to gentoo-prefix, but the system is really to old to be well-supported. As I know, the oldest version of CentOS Gentoo-prefix (libc version) supports is 5.6. Believe me, I have to work in a 4.3 environment... I have succeed one or two times to make gentoo-prefix-libc work, but gentoo-prefix-libc is not so helpful for development, not to mention deploy.

Then, my company begin to offer virtual develop machines, with root permission !!! yes!!! with root permission, I can easily get a ubuntu rootfs and chroot, enjoy it. Then the problem comes, how can I deploy my binary to other machine? Sometimes static-linking gives some help, but a heavy binary is not that elegant. And correctly static-linking all libraries is not a easy task, some libraries only release the shared-linking version.

So, why not just carry all the shared libraries with the binary, as well as the loader (ld-linux.so). Then I do some experiments and get it done. 

Actually, this is just an experiment, and there is no black magic but patch ELF with patchelf, set rpath and interpreter. The method is widely used by people. And deploying this way should not be used in serious circumstances.

# Usage
```
# in a dev machine with higher version of gcc compiler.
$ FRUIT_PATH=....
$ FRUIT_PATH/fruit some-binary
# copy the output tar, fruit.tar.gz, to a machine with lower version of gcc (glibc)
# in machine with low version gcc
$ tar xzf fruit.tar.gz
$ fruit/juice
# done
```
## instructions
The `fruit` script collects all shared libraries the binary dependences on, and create a tarball of them. The tarball also contains a patchelf (comes from ubuntu 16.04), as well as the shared libraries patchelf dependences on (manily libc++, libc).

The `juice` script just use the patchelf (with its ld-linux.so loader) to patch the binary, set the interpreter and rpath, and run the binary. The script will donothing if the binary has already been patched. When move or rename the package, the `juice` script will automatically repatch the binary.
