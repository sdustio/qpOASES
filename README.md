
### Get the code:

    git clone https://github.com/sdustio/serial.git

### Examples:

    cmake -DQPOASES_BUILD_EXAMPLES:BOOL=TRUE -DCMAKE_BUILD_TYPE:STRING=Debug -H$(pwd) -B$(pwd)/build
    cmake --build $(pwd)/build --target all

### Testing:
    cmake -DQPOASES_BUILD_TESTS:BOOL=TRUE -DCMAKE_BUILD_TYPE:STRING=Debug -H$(pwd) -B$(pwd)/build
    cmake --build $(pwd)/build --target all
    cd $(pwd)/build
    ctest
    ctest -T memcheck

### Install:

    cmake -DCMAKE_INSTALL_PREFIX:STRING=$HOME/.local -H$(pwd) -B$(pwd)/build
    cmake --build $(pwd)/build --target install

### Build the documentation:

    doxygen doxygen.config

