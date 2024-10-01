# alpentune

Alpentune is a C audio library, originally developed for [Brenta Engine](https://github.com/San7o/Brenta-Engine).

Currently only ALSA backend is in developement.

## Building

### cmake
```bash
cmake -Bbuild
cmake --build build -j 4
```
### meson
```bash
meson setup build
ninja -C build
```

## Testing
```
cmake -Bbuild -DALPENTUNE_BUILD_TESTS=ON
cmake --build build -j 4
./build/tests --help
```
The library uses [valFuzz](https://github.com/San7o/valFuzz) for testing
```c++
./build/tests              # run tests
./build/tests --fuzz       # run fuzzer
./build/tests --benchmark  # run benchmarks
```

## Documentation

The library uses doxygen for documentation, to build the html documentation run:
```
make docs
```
