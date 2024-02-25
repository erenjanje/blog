
# Programs
COMPILER := g++
RM := rm -f
MKDIR := mkdir -p
EMPTY_ECHO := echo
RUNNER :=

# Extensions
SRC_EXTENSION := .cpp
OBJ_EXTENSION := .o
DEPENDS_EXTENSION := .d
STATIC_LIB_EXTENSION := .a
DYNAMIC_LIB_EXTENSION := .so

# Helper functions
rwildcard=$(wildcard $1$2) $(foreach d,$(wildcard $1*),$(call rwildcard,$d/,$2))
makewithdirext=$(patsubst $2%$3,$4%$5,$1)
makeobj=$(call makewithdirext,$1,$(SRCDIR),$(SRC_EXTENSION),$(OBJDIR),$(OBJ_EXTENSION))
makedepend=$(call makewithdirext,$1,$(SRCDIR),$(SRC_EXTENSION),$(DEPENDDIR),$(DEPENDS_EXTENSION))
makestatic=$(call makewithdirext,$1,$(LIBDIR)lib,$(STATIC_LIB_EXTENSION),,)
makedynamic=$(call makewithdirext,$1,$(LIBDIR)lib,$(DYNAMIC_LIB_EXTENSION),,)
map=$(foreach e,$2,$(call $1,$e))

# Directories
SRCDIR := source/
INCDIR := $(SRCDIR)
LIBDIR := lib/
BUILDDIR := build/
OBJDIR := $(BUILDDIR)objects/
BINDIR := $(BUILDDIR)bin/
DEPENDDIR := depends/

# Files
SRCS := $(call rwildcard,$(SRCDIR),*$(SRC_EXTENSION))
STATIC_LIBS := $(call rwildcard,$(LIBDIR),*$(STATIC_LIB_EXTENSION))
DYNAMIC_LIBS := $(call rwildcard,$(LIBDIR),*$(DYNAMIC_LIB_EXTENSION))

# Processed files
OBJS := $(call map,makeobj,$(SRCS))
DEPENDS := $(call map,makedepend,$(SRCS))

# Compiler options
DEBUG := T
OPTIMIZATION_LEVEL := $(if $(DEBUG),g,3)
ENABLED_WARNINGS := all extra
DISABLED_WARNINGS := unused-parameter unknown-pragmas
ERRORS := $(ENABLED_WARNINGS) $(DISABLED_WARNINGS:%=no-%)
DEFINES := $(if $(DEBUG),DEBUG,RELEASE) GLFW_INCLUDE_NONE
FLAGS := openmp modules-ts PIC $(if $(DEBUG),sanitize=leak,)
STANDARD := c++20

# Linker options
LIBS := m glfw

# Make the options suitable for passing to compiler/flagize them
_DEBUG := $(if $(DEBUG),-g,)
_ERRORS := $(ERRORS:%=-W%)
_DEFINES := $(DEFINES:%=-D%)
_FLAGS := $(FLAGS:%=-f%)
_STANDARD := $(STANDARD:%=-std=%)

# Make the linker options suitable flags
FOUND_LIBS := $(call map,makestatic,$(STATIC_LIBS))
_ALL_LIBS := $(FOUND_LIBS) $(LIBS)
_LIBS := $(_ALL_LIBS:%=-l%)

CXXFLAGS := -I"./$(INCDIR)" -O$(OPTIMIZATION_LEVEL) $(_DEFINES) $(_STANDARD) $(_FLAGS) $(_ERRORS)
LDFLAGS := $(CXXFLAGS) -L"$(LIBDIR)" $(_LIBS)

EXECNAME := main
ARGS := 

run: build
	@echo RUNNING $(EXECNAME)
	@echo ===================
	@$(EMPTY_ECHO)
	@$(BINDIR)$(EXECNAME) $(ARGS)
.PHONY: run

debug:
	@echo $(FOUND_LIBS)
	@echo $(DEPENDS)
.PHONY: debug

build: $(OBJS)
	@echo LINKING $(EXECNAME)
	@$(COMPILER) $^ -o $(BINDIR)$(EXECNAME) $(LDFLAGS)
.PHONY: build

clean:
	@echo RM $(OBJS)
	@$(RM) $(OBJS)
.PHONY: clean

$(OBJDIR)%$(OBJ_EXTENSION) $(DEPENDDIR)%$(DEPENDS_EXTENSION): $(SRCDIR)%$(SRC_EXTENSION)
	@echo COMPILE $<
	@$(MKDIR) $(patsubst $(SRCDIR)%,$(OBJDIR)%,$(^D))
	@$(MKDIR) $(patsubst $(SRCDIR)%,$(DEPENDDIR)%,$(^D))
	@-$(COMPILER) $(CXXFLAGS) -MMD -MP -MF $(call makedepend,$<) -c $< -o $(call makeobj,$<)

init:
	@echo "Initializing Folders"
	@$(MKDIR) $(SRCDIR)
	@$(MKDIR) $(INCDIR)
	@$(MKDIR) $(OBJDIR)
	@$(MKDIR) $(LIBDIR)
	@$(MKDIR) $(BUILDDIR)
	@$(MKDIR) $(BINDIR)
	@$(MKDIR) $(DEPENDDIR)
	@echo "Initialization Completed"
.PHONY: init

-include $(DEPENDS)
