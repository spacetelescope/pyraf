// Obtain files from source control system.
if (utils.scm_checkout()) return

// Globals
PIP_INST = "pip install"
CONDA_INST = "conda install -y -q"
PY_SETUP = "python setup.py"
PYTEST_ARGS = "tests --basetemp=tests_output --junitxml results.xml"

matrix_python = ["2.7", "3.5", "3.6"]
matrix_astropy = ["2", "3"]
matrix = []

// Define each build configuration, copying and overriding values as necessary.

// RUN ONCE:
//    PyRAF is technically 2.7-only.
test27 = new BuildConfig()
test27.nodetype = "linux-stable"
test27.name = "test-suite"
test27.env_vars = ['PYRAF_NO_DISPLAY=1']
test27.build_cmds = ["conda config --add channels http://ssb.stsci.edu/astroconda",
                     "${CONDA_INST} python=2.7 iraf-all six",
                     "${CONDA_INST} --only-deps pyraf",
                     "${PY_SETUP} build_ext --inplace",
                     "${PY_SETUP} install"]
test27.test_cmds = ["with_env mkiraf -f xterm",
                    "with_env pytest ${PYTEST_ARGS}"]
test27.failedUnstableThresh = 1
test27.failedFailureThresh = 6
matrix += test27

// RUN ONCE:
//    Also test in PY3 but allow it to fail.
test36 = new BuildConfig()
test36.nodetype = "linux-stable"
test36.name = "test-suite-py3"
test36.env_vars = ['PYRAF_NO_DISPLAY=1']
test36.build_cmds = ["conda config --add channels http://ssb.stsci.edu/astroconda",
                     "${CONDA_INST} python=3.6 iraf-all six d2to1 stsci.distutils",
                     "${CONDA_INST} --only-deps pyraf",
                     "2to3 -w . &>/dev/null",
                     "${PY_SETUP} build_ext --inplace",
                     "${PY_SETUP} install"]
// NOTE: Remove "|| true" when it is no longer allowed to fail.
test36.test_cmds = ["with_env mkiraf -f xterm",
                    "with_env pytest ${PYTEST_ARGS} || true"]
// NOTE: Set threshold values to be same as test27 when it is no longer allowed to fail.
test36.failedUnstableThresh = 66
test36.failedFailureThresh = 666
matrix += test36

// RUN ONCE:
//    "sdist" is agnostic enough to work without any dependencies
sdist = new BuildConfig()
sdist.nodetype = "linux-stable"
sdist.name = "sdist"
sdist.build_cmds = ["${PY_SETUP} sdist"]
matrix += sdist

// TODO: Do we need this?
// Generate installation compatibility matrix
/*
for (python_ver in matrix_python) {
    for (astropy_ver in matrix_astropy) {
        // Astropy >=3.0 no longer supports Python 2.7
        if (python_ver == "2.7" && astropy_ver == "3") {
            continue
        }

        DEPS = "python=${python_ver} astropy=${astropy_ver}"

        install = new BuildConfig()
        install.nodetype = "linux-stable"
        install.name = "install ${DEPS}"
        install.build_cmds = ["${CONDA_INST} ${DEPS}",
                              "${PY_SETUP} egg_info",
                              "${PY_SETUP} install",
                              "pyraf --help",
                              "pyraf --version"]
        matrix += install
    }
}
*/

// TODO: Run real tests on dev build
// Generate dev build compatibility matrix
/*
for (python_ver in matrix_python) {
    PIP_DEPS = ["git+https://github.com/spacetelescope/stsci.tools#egg=stsci.tools --upgrade --no-deps",
                "numpy --upgrade --no-deps",
                "ipython --upgrade",
                "matplotlib --upgrade"]

    install_pypi = new BuildConfig()
    install_pypi.nodetype = "linux-stable"
    install_pypi.name = "install ${python_ver}-DEV"

    // 2.7 doesn't work with Astropy-dev, so skip it.
    if (python_ver == "2.7") {
        PIP_DEPS += ["'astropy<3.0' --force --upgrade --no-deps"]
    }
    else {
        PIP_DEPS += ["Cython",
                     "git+https://github.com/astropy/astropy.git#egg=astropy --upgrade --no-deps"]
    }

    for (dep in PIP_DEPS) {
        install_pypi.build_cmds += "${PIP_INST} ${dep}"
    }
    install_pypi.build_cmds += ["${PY_SETUP} egg_info",
                                "${PY_SETUP} install",
                                "pyraf --help",
                                "pyraf --version"]
    matrix += install_pypi
}
*/

// Iterate over configurations that define the (distibuted) build matrix.
// Spawn a host of the given nodetype for each combination and run in parallel.
utils.run(matrix)
