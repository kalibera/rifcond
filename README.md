This is a report from checking for packages executing `if` or `while`
statement on a condition of length bigger than one. This is now still a
warning in R, but as it almost always is caused by error in the program, it
will soon be turned into an error.

The report is based on running R-devel 74151 with environment variables
`R_KEEP_PKG_SOURCE=yes` (to keep source references in packages) and
`_R_CHECK_LENGTH_1_CONDITION_=package:_R_CHECK_PACKAGE_NAME_` (to fail on
condition of length bigger than one, but only when the if/while statement is
in the package being currently checked, so the risk of false alarms is very
small).

R has been modified slightly so that it calls `R_Suicide` (aborts
unconditionally) and reports diagnostic information to help debugging.  Only
one instance of the problem per checked package would be covered by this
experiment.  One can use `_R_CHECK_LENGTH_1_CONDITION_=true` with the
current R-devel to debug the code interactively and to detect further
similar errors in a package.
