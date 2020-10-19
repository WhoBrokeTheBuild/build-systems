# Targets that don't create files are called phony, so we declare 
# them explicitly
.PHONY: all clean install run gdb test

# Ensure directories exist
$(shell mkdir -p src/ obj/ bin/)

# Set the standard compiler and linker flags
CXXFLAGS += -g -Wall $(shell pkg-config --cflags sdl2)
LDFLAGS  += $(shell pkg-config --libs-only-L sdl2)
LDLIBS   += $(shell pkg-config --libs-only-l sdl2)

PREFIX ?= /usr/local

# Configure variables used by our rules
TARGET = bin/example

# Gather our source files, and a list of object file counterparts
SOURCES = $(wildcard src/**/*.cpp src/*.cpp)
OBJECTS = $(patsubst src/%.cpp, obj/%.o, $(SOURCES))

# Gather our test source files, and create a target for each
TEST_SOURCES = $(wildcard tests/*_test.cpp)
TEST_TARGETS = $(patsubst tests/%.cpp, bin/%, $(TEST_SOURCES))

# The general rule for building all .cpp files into .o files
# The proper way to do this would involve some flags to build a
# dependency file, usually a .d, which could track which header
# files have changed as well.
obj/%.o: src/%.cpp
	$(CXX) $(CXXFLAGS) -c -o $@ $<

# The rule for building our main target
$(TARGET): $(OBJECTS)
	$(CXX) $(LDFLAGS) -o $@ $^ $(LDLIBS)

all: $(TARGET)

# Remove all targets and objects
clean:
	rm -f $(TARGET) $(OBJECTS) $(TEST_TARGETS)

# Install into the proper directory, as pointed to by
# the standard DESTDIR/PREFIX variables
install: $(TARGET)
	install -d $(DESTDIR)/$(PREFIX)/bin/
	install $(TARGET) $(DESTDIR)/$(PREFIX)/bin/

# Run our target
run: $(TARGET)
	./$(TARGET)

# Run our target, but with gdb
gdb: $(TARGET)
	gdb --args $(TARGET)

# Run our target, but with valgrind
valgrind: $(TARGET)
	valgrind $(TARGET)

# The rule to build each test
# In order to make this useful, you would want to build in the
# objects from the main target, or otherwise link agains the
# code you would be testing. For the point of this example,
# we're going to skip over that.
bin/%_test: tests/%_test.cpp
	$(CXX) -I src -o $@ $<

# Build and run all of our tests
test: $(TEST_TARGETS)
	$(addsuffix ;,$(TEST_TARGETS))