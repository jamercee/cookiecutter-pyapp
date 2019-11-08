# ----------------------------------------------------------------------------
# Define system macros
# ----------------------------------------------------------------------------
ifeq ($(OS), Windows_NT)
	OSTYPE := Windows
	PYTHON := python.exe
	PYLINT := pylint.exe
	FLAKE := flake8.exe
else
	OSTYPE := $(shell uname)
	PYTHON := python
	PYLINT := pylint
	FLAKE := flake8
endif

PYARCH := $(shell $(PYTHON) -c "import platform; print(platform.architecture()[0])")

# ----------------------------------------------------------------------------
# Define $(BROWSER) for reading documentation with web browser
# ----------------------------------------------------------------------------
define BROWSER_PYSCRIPT
import os, webbrowser, sys
try:
	from urllib import pathname2url
except:
	from urllib.request import pathname2url

webbrowser.open("file://" + pathname2url(os.path.abspath(sys.argv[1])))
endef
export BROWSER_PYSCRIPT
BROWSER := python -c "$$BROWSER_PYSCRIPT"

# ----------------------------------------------------------------------------
# Rule to compile python *.py -> *.pylint
# ----------------------------------------------------------------------------
%.pylint:	%.py
	@echo Check $<
	@$(PYLINT) -rn -sn --rcfile pylint.rc $<
	@$(FLAKE) $<
	@touch $@

%.pln: %.pyx
	@echo Check $<
	@$(PYLINT) -rn --rcfile pylint.rc $<
	@touch $@

# ----------------------------------------------------------------------------

SRCS := $(wildcard src/*.py) $(wildcard tests/*.py)
TGTS := $(patsubst %.py, %.pylint,$(SRCS))

.PHONY: clean clean-build clean-docs clean-pyc comp debug docs help 

TESTX := tests.test_targetx

all: comp

help:
	@echo "comp - perform static analysis (default target)"
	@echo "tests - run unittests"
	@echo "docs - generate html documentation"
	@echo "dist - build a source distribution"
	@echo "build - build everything needed to install"
	@echo "install - install package to site-packages"
	@echo "viewdocs - generate & view html documentation"
	@echo "clean - remove all built components"
	@echo "clean-build - remove all built outputs"
	@echo "clean-pyc - remove Python file artifacts"
	@echo "debug - generate Makefile diagnostic output"

comp: $(TGTS)

# Run all tests with nose, exclude 'slow' and 'bigdisk' test-cases
ntests: comp
	@$(PYTHON) setup.py nosetests -A "not slow and not bigdisk"

# Run all tests using python's unittest module
ptests: comp
	@$(PYTHON) setup.py test

testx: comp ntestx ptestx

# Run specified test-case with nose
ntestx: comp
	@$(PYTHON) setup.py nosetests --verbosity=3 \
		--tests $(TESTX)

# Run specified test-case with pythons unittest module
ptestx: comp
	@$(PYTHON) setup.py test -s $(TESTX)

docs:
	@$(MAKE) -C docs html

viewdocs: docs
	$(BROWSER) docs/_build/html/index.html

dist: tests docs
	@$(PYTHON) setup.py sdist

install: tests docs
	@$(PYTHON) setup.py install

clean: clean-build clean-pyc clean-docs

clean-build:
	-rm -fr build/ dist/ .eggs/
	find . -name '*.egg-info' -exec rm -fr {} +
	find . -name '*.egg' -exec rm -fr {} +

clean-pyc:
	find . -name '*.py[cod]' -exec rm -f {} +
	find . -name '*.pylint' -exec rm -f {} +
	find . -name '*.pln' -exec rm -f {} +
	find . -name '~*' -exec rm -f {} +
	find . -name '__pycache__' -exec rm -fr {} +

clean-docs:
	$(MAKE) -C docs clean
	
debug:
	@echo OSTYPE:: $(OSTYPE)
	@echo PYTHON:: $(PYTHON)
	@echo PYLINT:: $(PYLINT)
	@echo FLAKE:: $(FLAKE)
	@echo Source files found::
	@echo $(SRCS)
	@echo Targets to build::
	@echo $(TGTS)
