# native-kernel
AUR package which rebuild latest kernel with `-march=native -mtune=native` and LLVM ThinLTO

# Build settings 
By-default script gets Kconfig from your current kernel, but script support custom presets via environment variables, you can configure your kernel with
```shell
_menuconfig=1 makepkg -si # or 
_menuconfig=1 yay -S native-kernel 
```

Also you can use GCC instead of clang (but witout LTO) 
```shell
_CLANG=0 makepkg -si
```

Verbose build: 
```shell
_verbose=1 makepkg -si 
```

And change linker: 
```shell 
_linker=ld.lld makepkg -si 
```

ofc you also can combine flags:

```shell 
_CLANG=1 _menuconfig=1 _linker=ld.bfd makepkg -si 
```

Enjoy your optimized linux!
