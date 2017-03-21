This is a report from running a modified version of R-devel 72354. R was
modified to abort when `if` is executed with a condition that has length
greater than 1. In older version of R this is allowed, but one gets a
warning and the first element is used. In recent versions of R one may
choose to get an error instead, via environment variable
`_R_CHECK_LENGTH_1_CONDITION_`.

The modified version that was run here used `R_Suicide` (aborts
unconditionally) and reports diagnostic information to help debugging.  This
information will end up in some output file, depending on which test/example
is running (and a part of it that fits will appear in `00check.log`).  This
is the code that produces the report

```c
if (length(s) > 1) {
  R_OutputCon = 2;
  R_ErrorCon = 2;
  REprintf(" ----------- FAILURE REPORT -------------- \n");
  REprintf(" --- srcref --- \n");
  SrcrefPrompt("", R_getCurrentSrcref());
  REprintf("\n");
  REprintf(" --- call (function) --- \n");
  PrintValue(R_GlobalContext->call);
  REprintf(" --- stacktrace ---\n");
  printwhere();
  REprintf(" --- value of length: %d type: %s ---\n", length(s), type2char(TYPEOF(s)));
  PrintValue(s);
  REprintf(" --- function --- \n");
  if (R_GlobalContext->callfun != NULL && TYPEOF(R_GlobalContext->callfun) == CLOSXP)
    PrintValue(R_GlobalContext->callfun);
  REprintf(" --- function (body) search ---\n");
  if (R_GlobalContext->callfun != NULL && TYPEOF(R_GlobalContext->callfun) == CLOSXP)
  findFunctionForBody(R_ClosureExpr(R_GlobalContext->callfun));
  REprintf(" ----------- END FAILURE REPORT -------------- \n");
  R_Suicide("the condition has length > 1 and only the first element will be used XXXXXX");
}
```
Only one instance of the problem per checked package would be covered by
this experiment. One can then use `_R_CHECK_LENGTH_1_CONDITION_=true` with the
current R-devel or possibly the code above to check for more problems after
fixing the reported one.

The outputs here are in directories named after the packages that failed,
which are not always the packages at fault (packages that include the
problematic call to `if`). This is clear from the outputs, e.g.

```
funcy.Rcheck/funcy-Ex.Rout
```

is an error report from a failure of package `funcy`.  This report would
appear in file `funcy-Ex.Rout`, but for simplicity in the outputs shown
here, everything but the report is removed.  The report in `funcy-Ex.Rout`
is

```
 --- srcref --- 
 at /tmp/RtmpEPvTZA/R.INSTALL21d932b797fc5/sm/R/regression.r#1055: 
 --- call (function) --- 
sm.weight2(x, opt$eval.points, h, weights = weights, options = opt)
 --- stacktrace ---
where 1 at /tmp/RtmpEPvTZA/R.INSTALL21d932b797fc5/sm/R/regression.r#272: sm.weight2(x, opt$eval.points, h, weights = weights, options = opt)
where 2: sm.regression.2d(x, y, h, model, weights, rawdata, options = opt)
where 3: smoothFct2D(x = cov.timeIn, y = cov.raw, h = hcov, eval.points = cov.timeOut, 
    weights = rep(1, length(cov.raw)), poly.index = c(1, 2), 
    eval.grid = FALSE, display = "none")
where 4: withCallingHandlers(expr, warning = function(w) invokeRestart("muffleWarning"))
where 5 at /tmp/RtmpMc7x3O/R.INSTALL1d64e10850861/funcy/R/funPrinComp.R#127: suppressWarnings(smoothFct2D(x = cov.timeIn, y = cov.raw, h = hcov, 
    eval.points = cov.timeOut, weights = rep(1, length(cov.raw)), 
    poly.index = c(1, 2), eval.grid = FALSE, display = "none")$estimate)
where 6 at /tmp/RtmpMc7x3O/R.INSTALL1d64e10850861/funcy/R/funPrinComp.R#20: fpc(Yin = Yin, Tin = Tin, Tout = NULL, isobs = isobs, dimBase = dimBase, 
    fpcCtrl = fpcCtrl, reg = reg)
where 7: fpca(Data(ds))

 --- value of length: 2 type: logical ---
[1] FALSE FALSE
 --- function --- 
function (x, eval.points, h, cross = FALSE, weights = rep(1, nrow(x)),
    options = list()) {

    # code of the function that was active when the if statement ran
    # (excluded here)
}
<bytecode: 0x1aeda510>
<environment: namespace:sm>
 --- function (body) search ---
Function sm.weight2 in namespace sm has this body.
```
One can see from the report that the problematic `if` statement is in
package `sm` and the source file with the `if` statement is `regression.r`, the
line is `1055` and it is in function `sm.weight2`.  The value for the
condition was a logical vector of 2 values `c(FALSE,FALSE)`.  One can also
see the R stacktrace.

One then has to look into `regression.r` to see that the problem is because
the option `poly.index` is not scalar. One can guess from the stacktrace shown
here that the options actually comes from a call to `smoothFct2D` inside
`funPrinComp.R` from `funcy`, at line `127`. From the [documentation of
`sm`](https://cran.r-project.org/web/packages/sm/sm.pdf) it seems that
`poly.index` should be a scalar value of `0` or `1`. 

To validate this it is probably best to run the example again with
`_R_CHECK_LENGTH_1_CONDITION_=true` and use the R debugger.

```
env _R_CHECK_LENGTH_1_CONDITION_=true ./bin/R CMD check ./down/funcy_0.8.6.tar.gz 
```

reproduces the error. A faster way to reproduce is just running the example

```
env _R_CHECK_LENGTH_1_CONDITION_=true ../bin/R --vanilla < funcy-Ex.R
```

so one can also run the example in an interactive session and use
`options(error=recover)` to start the debugger where the problem happens. 
The `opt$poly.index` in `sm.weight2` is the same one (even by reference) as
in `smoothFct2D`, and it is `c(1,2)`.
