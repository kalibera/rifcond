This is a report from checking for packages executing `if` or `while`
statement on a condition of length bigger than one. This is now still a
warning in R, but as it almost always is caused by error in the program, it
will soon be turned into an error.

The report is based on running R-devel 75396 with environment variables
`R_KEEP_PKG_SOURCE=yes` (to keep source references in packages) and
`_R_CHECK_LENGTH_1_CONDITION_=package:_R_CHECK_PACKAGE_NAME_,abort,verbose`
(to fail on condition of length bigger than one, but only when the if/while
statement is in the package being currently checked, so the risk of false
alarms/false identification of a package to fix, is very small).

With the `abort` option, R aborts unconditionally and reports diagnostic
information to help debugging, but this means that only one instance of the
problem per a running R instance would be covered by this experiment.  One
can set the environment variables in the same way with the current R-devel
to to detect further similar errors in a package.  Without `abort`, one
would get a normal runtime R error, allowing interactive debugging if
needed.
