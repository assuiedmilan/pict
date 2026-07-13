Pairwise Independent Combinatorial Testing
==========================================

PICT generates test cases and test configurations. With PICT, you can generate tests that are more effective than manually generated tests and in a fraction of the time required by hands-on test case design.

PICT runs as a command line tool. Prepare a model file detailing the parameters of the interface (or set of configurations, or data) you want to test. PICT generates a compact set of parameter value choices that represent the test cases you should use to get comprehensive combinatorial coverage of your parameters.

For instance, if you wish to create a test suite for partition and volume creation, the domain can be described by the following parameters: **Type**, **Size**, **File system**, **Format method**, **Cluster size**, and **Compression**. Each parameter has a limited number of possible values, each of which is determined by its nature (for example, **Compression** can only be **On** or **Off**) or by the equivalence partitioning (such as **Size**).

    Type:          Single, Span, Stripe, Mirror, RAID-5
    Size:          10, 100, 500, 1000, 5000, 10000, 40000
    Format method: Quick, Slow
    File system:   FAT, FAT32, NTFS
    Cluster size:  512, 1024, 2048, 4096, 8192, 16384, 32768, 65536
    Compression:   On, Off

There are thousands of possible combinations of these values. It would be  difficult to test all of them in a reasonable amount of time. Instead, we settle on testing all possible pairs of values. For example, **{Single, FAT}** is one pair, **{10, Slow}** is another; one test case can cover many pairs. Research shows that testing all pairs is an effective alternative to exhaustive testing and much less costly. It will provide very good coverage and the number of test cases will remain manageable.

# More information

See **[doc/pict.md](https://github.com/Microsoft/pict/blob/main/doc/pict.md)** for detailed documentation on PICT and http://pairwise.org has details on this testing methododology. 

The most recent **pict.exe** is available at https://github.com/microsoft/pict/releases/.

# Contributing

PICT consists of the following projects:
 * **api**: The core combinatorial engine, built as the static library **pict_api** and as the shared library **pict_shared** (`pict.dll` / `libpict.dylib` / `libpict.so`) whose exported C API (see **api/pictapi.h**) can be consumed from other languages, e.g. C# via `[DllImport("pict")]`,
 * **cli**: The PICT command-line tool, built as the executable **pict_cli**.

The **clidll**, **api-usage**, and **clidll-usage** directories contain legacy sample/wrapper sources and are not part of the build.

## Building and testing (Windows, macOS, Linux, *BSD)
PICT uses CMake as its only build system. Assuming an installation of CMake and a C++17 toolchain (MSVC, Clang, or GCC), the following set of commands will build PICT and run the tests from the directory `build`:
```
> cmake -DCMAKE_BUILD_TYPE=Release -S . -B build
> cmake --build build --config Release
> ctest --test-dir build -C Release
```

On Windows, `cmake -S . -B build` generates a Visual Studio solution in **build/PICT.sln** which you can open in Visual Studio 2022 instead of building from the command line.

The **test** folder contains all that is necessary to test PICT. You need Perl to run the tests (on Windows, e.g. [Strawberry Perl](https://strawberryperl.com/)); CMake registers the test suite only when Perl is found.

The test script produces a log: **dbg.log** or **rel.log** for the Debug and Release bits respectively. Compare them with their committed baselines and make sure all the differences can be explained.

>There are tests which randomize output which typically make it different on each run. These results should be masked in the baseline but currently aren't.

## Debugging

Most commonly, you will want to debug the command-line tool. Start in the **pict_cli** target, **cli/pict.cpp** file. You'll find **wmain** routine there which would be a convenient place to put the very first breakpoint.

## PICT as a container

To build a container image with PICT, just execute

    podman build --layers=true -t pict:latest .

Once built, you can run it with a sample model as follows

    podman run -it --rm -v ./doc/sample-models:/var/pict:Z pict:latest create_volume.txt

To use your own models, please execute

    podman run -it --rm -v ./<local-dir>:/var/pict:Z pict:latest <your-model-file> [<pict-options>]

