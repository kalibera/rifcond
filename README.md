This is a report from checking for packages executing `if` or `while`
statement on a condition of length bigger than one. This is now still a
warning in R, but as it almost always is caused by error in the program, it
will soon be turned into an error.

The reports in `outputs` are based on running R-devel 75396 with environment
variables `R_KEEP_PKG_SOURCE=yes` (to keep source references in packages)
and `_R_CHECK_LENGTH_1_CONDITION_=package:_R_CHECK_PACKAGE_NAME_,abort,verbose`
(to fail on condition of length bigger than one, but only when the if/while
statement is in the package being currently checked, so the risk of false
alarms/false identification of a package to fix, is very small). The outputs
are from recent CRAN packages and from BIOC 3.7, package versions are
detailed in the `README` file. The reports in `outputs_38` are for tests
with BIOC 3.8.

The reports in `outputs_all` use `R_KEEP_PKG_SOURCE=yes` and
`_R_CHECK_LENGTH_1_CONDITION_=abort,verbose`, which means that R fails on
condition of length bigger than one no matter from which package is the
problematic code. The outputs are from recent CRAN packages and from BIOC
3.7, packages versions are again detailed in the `README` file.

With the `abort` option, R aborts unconditionally and reports diagnostic
information to help debugging, but this means that only one instance of the
problem per a running R instance would be covered by this experiment.  One
can set the environment variables in the same way with the current R-devel
to to detect further similar errors in a package.  Without `abort`, one
would get a normal runtime R error, allowing interactive debugging if
needed.

The reports should be fairly self-explanatory.  Look for a line containing
`FAILURE REPORT` near the end of an output file (but note that some output
files cover more than one R session, so they may have multiple failure
reports).  The `srcref` field gives file name and line number for the
if/while statement, the `call from argument` field gives the if/while
statement expression, the `value` field gives the type and length of the
actual value passed to the condition (so, the value that is of length bigger
than one).
