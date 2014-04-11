
# Source code
# Note that BuildVersion is not forced to be rebuilt here - so incremental
# makes in this directory will endlessly use the same BuildVersion.
main_javas = $(wildcard src/main/water/*java src/main/water/*/*java)
test_javas = $(wildcard src/test/water/*java)
main_classes = $(patsubst src/main/%java,build/classes/%class,$(main_javas))
test_classes = $(patsubst src/test/%java,build/classes/%class,$(test_javas))
classes = $(main_classes) $(test_classes)

# All the libraries; these are downloaded by gradle and slapped into the lib/ directory.
libs = $(wildcard ../lib/*jar)
jars = $(subst $(space),$(SEP),$(libs))

# Fun Args to javac.  Mostly limit to java6 source definitions, and fairly
# agressive lint warnings.
JAVAC_ARGS = -g -source 1.7 -target 1.7 -XDignore.symbol.file -Xlint:all -Xlint:-deprecation -Xlint:-serial -Xlint:-rawtypes -Xlint:-unchecked

# literal space
space :=
space +=

# jar-file seperator
ifeq (,$findstring cygwin,$(MAKE_HOST))  
# linux
SEP = :
else   
# windows
SEP = ;
endif

# Default build: the h2o-core jar file
default: build/h2o-core.jar

# Just the classes, no jarring step
classes: $(classes)

check: $(classes)
	java -ea -cp "build/classes;$(jars)" org.junit.runner.JUnitCore water.KVTest

# Compile just the out-of-date files
$(main_classes): build/classes/%class: src/main/%java
	@echo "compiling " $?
	@[ -d build/classes ] || mkdir -p build/classes
	@javac $(JAVAC_ARGS) -cp "build/classes$(SEP)$(jars)" -sourcepath src/main -d build/classes $(main_javas)

$(test_classes): build/classes/%class: src/test/%java
	@echo "compiling " $?
	@[ -d build/classes ] || mkdir -p build/classes
	@javac $(JAVAC_ARGS) -cp "build/classes$(SEP)$(jars)" -sourcepath src/main -d build/classes $(test_javas)

build/classes/water/init/BuildVersion.class: ../build/BuildVersion.java
	@echo "compiling " $?
	@javac $(JAVAC_ARGS) -sourcepath src/main -d build/classes $?

# Build the h2o-core.jar file
# All included jars are unpacked into a flat structure, then repacked into One Jar
# Name collisions amongst packages are quietly ignored.
# H2O names win over all other names.
build/h2o-core.jar: $(classes) build/classes/water/init/BuildVersion.class
	@mkdir -p build/tmp
	@(cd build/tmp; for d in $(libs); do ( jar xf ../../$$d ); done )
	@jar -cfm build/h2o-core.jar manifest.txt -C build/classes . -C build/tmp .
	@rm -rf build/tmp


.PHONY: clean
clean:
	rm -rf build


# Called "conf" here, after auto-conf, but really just asks each sub-make to list tools
.PHONY: conf
conf:
	@echo $(CURDIR) requires java, jar
	java -version
	which jar