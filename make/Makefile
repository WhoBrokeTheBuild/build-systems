# Targets that don't create files are called phony, so we declare 
# them explicitly
.PHONY: all clean install run gdb valgrind test

# Set the default install location
PREFIX ?= /usr/local

# Set the standard compiler and linker flags
CXXFLAGS += -g -Wall $(shell pkg-config --cflags sdl2)
LDFLAGS  += $(shell pkg-config --libs-only-L sdl2)
LDLIBS   += $(shell pkg-config --libs-only-l sdl2)

# Set compiler and linker flags specific to the executable
EXE_CXXFLAGS = $(CXXFLAGS) -Iinclude
EXE_LDFLAGS  = $(LDFLAGS) -Llib
EXE_LDLIBS   = $(LDLIBS) -lexample

# Set compiler flags specific to the executable
LIB_CXXFLAGS = $(CXXFLAGS) -Iinclude

# Configure our target files
EXE_TARGET = bin/example
LIB_TARGET = lib/libexample.a

EXE_SRCDIR = src/example
EXE_OBJDIR = obj/example
LIB_SRCDIR = src/libexample
LIB_OBJDIR = obj/libexample

# Gather our source files, and a list of object file counterparts
EXE_SOURCES = $(wildcard $(EXE_SRCDIR)/**/*.cpp $(EXE_SRCDIR)/*.cpp)
EXE_OBJECTS = $(patsubst $(EXE_OBJDIR)/%.cpp, $(EXE_OBJDIR)/%.o, $(EXE_SOURCES))
LIB_SOURCES = $(wildcard $(LIB_SRCDIR)/**/*.cpp $(LIB_SRCDIR)/*.cpp)
LIB_OBJECTS = $(patsubst $(LIB_OBJDIR)/%.cpp, $(LIB_OBJDIR)/%.o, $(LIB_SOURCES))

# Gather our test source files, and create a target for each
TEST_SOURCES = $(wildcard tests/*_test.cpp)
TEST_TARGETS = $(patsubst tests/%.cpp, bin/%, $(TEST_SOURCES))

# The general rules for building all .cpp files into .o files
# The proper way to do this would involve some flags to build a
# dependency file, usually a .d, which could track which header
# files have changed as well.
$(EXE_OBJDIR)/%.o: $(EXE_SRCDIR)/%.cpp
	$(CXX) $(EXE_CXXFLAGS) -c -o $@ $<

$(LIB_OBJDIR)/%.o: $(LIB_SRCDIR)/%.cpp
	$(CXX) $(LIB_CXXFLAGS) -c -o $@ $<

# The rule for building our executable target
$(EXE_TARGET): $(LIB_TARGET) $(EXE_OBJECTS)
	$(CXX) $(EXE_LDFLAGS) -o $@ $(EXE_OBJECTS) $(EXE_LDLIBS)

# The rule for building our library target
$(LIB_TARGET): $(LIB_OBJECTS)
	$(AR) rcs $@ $^

all: $(EXE_TARGET) $(LIB_TARGET)

# Remove all targets and objects
clean:
	rm -f $(EXE_TARGET) $(LIB_TARGET) $(TEST_TARGETS) $(EXE_OBJECTS) $(LIB_OBJECTS)

# Install into the proper directory, as pointed to by
# the standard DESTDIR/PREFIX variables
install: $(EXE_TARGET) $(LIB_TARGET)
	install -d $(DESTDIR)/$(PREFIX)/bin/
	install $(EXE_TARGET) $(DESTDIR)/$(PREFIX)/bin/
	install -d $(DESTDIR)/$(PREFIX)/lib/
	install $(LIB_TARGET) $(DESTDIR)/$(PREFIX)/lib/

# Run our executable target
run: $(EXE_TARGET)
	./$(EXE_TARGET)

# Run our executable target, but with gdb
gdb: $(EXE_TARGET)
	gdb --args $(EXE_TARGET)

# Run our executable target, but with valgrind
valgrind: $(EXE_TARGET)
	valgrind $(EXE_TARGET)

# The rule to build each test, linking them against our library target
bin/%_test: tests/%_test.cpp $(LIB_TARGET)
	$(CXX) $(EXE_CXXFLAGS) $(EXE_LDFLAGS) -o $@ $< $(EXE_LDLIBS)

# Build and run all of our tests
test: $(TEST_TARGETS)
	$(addsuffix ;,$(TEST_TARGETS))