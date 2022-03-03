# utl-using-varable-names-in-your-data-to-create-new-columns
Using varable names in your data to create new columns
    %let pgm=utl-using-varable-names-in-your-data-to-create-new-columns;

     Using varable names in your data to create new columns

         1. SAS datastep  (simplification of stackoverflow solutions?)
         2. SAS SQL
         3. SAS SQl arrays
         4. R transpose  (tidyverse - gather can be uses to go fat to skinny))
         5. R SQL
         6, Python SQL (too messy https://tinyurl.com/5n9x9b5j
            use sql (you can use the stack function make a fat dataframe skinny)

    github
    https://tinyurl.com/4k7ansmz
    https://github.com/rogerjdeangelis/utl-using-varable-names-in-your-data-to-create-new-columns

    StackOverflow
    https://tinyurl.com/3edzepk4
    https://stackoverflow.com/questions/71332082/sas-how-to-name-the-column-with-the-value-of-other-variable

    /*                   _
    (_)_ __  _ __  _   _| |_
    | | `_ \| `_ \| | | | __|
    | | | | | |_) | |_| | |_
    |_|_| |_| .__/ \__,_|\__|
            |_|
    */

    libname sd1 "d:/sd1";

    options validvarname=upcase;

    * normalize - make long and skinny;
    data sd1.have;
      infile cards4;
      input (INDIC code1 code2 code3)($);
      array varnam code:;
      do over varnam;
         nam = varnam;
         val = 1;
         output;
      end;
      keep INDIC nam val;
    cards4;
    A P1 P2 P3
    B P1 P3 P4
    C P2 P4 N1
    ;;;;
    run;quit;

    /*************************************************************/
    /*                                                           */
    /*                                                           */
    /* Up to 40 obs SD1.HAVE total obs=9 03MAR2022:11:30:50      */
    /*                                                           */
    /*                             | RULES for INDIC=A           */
    /*                             |                             */
    /* Obs    INDIC    NAM    VAL  | INDIC  P1 P2 P3 P4 N1       */
    /*                             |                             */
    /*  1       A      P1      1   |    A    1  1  1  0  0       */
    /*  2       A      P2      1   |                             */
    /*  3       A      P3      1   |                             */
    /*                             |                             */
    /*  4       B      P1      1   |                             */
    /*  5       B      P3      1   |                             */
    /*  6       B      P4      1   |                             */
    /*  7       C      P2      1   |                             */
    /*  8       C      P4      1   |                             */
    /*  9       C      N1      1   |                             */
    /*                                                           */
    /*************************************************************/

    /*           _               _
      ___  _   _| |_ _ __  _   _| |_
     / _ \| | | | __| `_ \| | | | __|
    | (_) | |_| | |_| |_) | |_| | |_
     \___/ \__,_|\__| .__/ \__,_|\__|
                    |_|
    */

    /*************************************************************/
    /*                                                           */
    /* Up to 40 obs WORK.WANT total obs=3 03MAR2022:11:39:48     */
    /*                                                           */
    /* Obs    INDIC    P1    P2    P3    P4    N1                */
    /*                                                           */
    /*  1       A       1     1     1     0     0                */
    /*  2       B       1     0     1     1     0                */
    /*  3       C       0     1     0     1     1                */
    /*                                                           */
    /*************************************************************/

    /*         _       _   _
     ___  ___ | |_   _| |_(_) ___  _ __  ___
    / __|/ _ \| | | | | __| |/ _ \| `_ \/ __|
    \__ \ (_) | | |_| | |_| | (_) | | | \__ \
    |___/\___/|_|\__,_|\__|_|\___/|_| |_|___/
         _       _            _
      __| | __ _| |_ __ _ ___| |_ ___ _ __
     / _` |/ _` | __/ _` / __| __/ _ \ `_ \
    | (_| | (_| | || (_| \__ \ ||  __/ |_) |
     \__,_|\__,_|\__\__,_|___/\__\___| .__/
                                     |_|
    */

    options missing='0'; * note missing will be numeric os;

    proc transpose data = sd1.have out = want(drop = _:);
       by INDIC;
       id nam;
       var val;
    run;quit;

    /*                         _
     ___  __ _ ___   ___  __ _| |
    / __|/ _` / __| / __|/ _` | |
    \__ \ (_| \__ \ \__ \ (_| | |
    |___/\__,_|___/ |___/\__, |_|
                            |_|
    */

    proc sql;
      create
         table want as
      select
          INDIC
         ,max(p1) as p1
         ,max(p2) as p2
         ,max(p3) as p3
         ,max(p4) as p4
         ,max(n1) as n1
      from
         (select
            INDIC
           ,case when (nam="P1") then "1" else "0" end as P1
           ,case when (nam="P2") then "1" else "0" end as P2
           ,case when (nam="P3") then "1" else "0" end as P3
           ,case when (nam="P4") then "1" else "0" end as P4
           ,case when (nam="N1") then "1" else "0" end as N1
          from
            have
         )
     group
        by INDIC
    ;quit;

    /*                         _
     ___  __ _ ___   ___  __ _| |   __ _ _ __ _ __ __ _ _   _ ___
    / __|/ _` / __| / __|/ _` | |  / _` | `__| `__/ _` | | | / __|
    \__ \ (_| \__ \ \__ \ (_| | | | (_| | |  | | | (_| | |_| \__ \
    |___/\__,_|___/ |___/\__, |_|  \__,_|_|  |_|  \__,_|\__, |___/
                            |_|                         |___/
    */

    proc sql;
      select
        distinct
          nam
        into :_n1-
      from
        sd1.have
    ;quit;

    %let _nn=&sqlobs;

    proc sql;
      create
         table want as
      select
          INDIC
         ,%do_over(_n,phrase=max(?) as ?,between=comma)
      from
         (select
            INDIC
            ,%do_over(_n, phrase=case when (nam="?") then "1" else "0" end as ?,between=comma)
          from
            sd1.have
         )
     group
        by INDIC
    ;quit;


    IF YOU WANT THE GENERATED CODE;

    %utlnopts;  * so just put statements go to log;

    data _null_;
       %do_over(_n,phrase=%str(put ",max(?) as ?";))
    run;quit;

    data _null_;
       %do_over(_n, phrase=%str(put "case when (nam='?') then '1' else '0' end as ?";));
    run;quit;
    %utlopts;

    LOG

    ,max(N1) as N1
    ,max(P1) as P1
    ,max(P2) as P2
    ,max(P3) as P3
    ,max(P4) as P4

    case when (nam='N1') then '1' else '0' end as N1
    case when (nam='P1') then '1' else '0' end as P1
    case when (nam='P2') then '1' else '0' end as P2
    case when (nam='P3') then '1' else '0' end as P3
    case when (nam='P4') then '1' else '0' end as P4

    /*      _   _     _
     _ __  | |_(_) __| |_   ___   _____ _ __ ___  ___
    | `__| | __| |/ _` | | | \ \ / / _ \ `__/ __|/ _ \
    | |    | |_| | (_| | |_| |\ V /  __/ |  \__ \  __/
    |_|     \__|_|\__,_|\__, | \_/ \___|_|  |___/\___|
                        |___/
    */

    %utl_submit_r64('
      library(haven);
      library(tidyverse);
      library(SASxport);
      have<-read_sas("d:/sd1/have.sas7bdat");
      want <- as.data.frame(spread(data = have, key = NAM, value = VAL));
      want[is.na(want)] <- 0;
      want;
      write.xport(want,file="d:/xpt/wantXpo.xpt");
    ');

    libname xpt xport "d:/xpt/wantXpo.xpt";

    proc print data=xpt.want;
    run;quit;

    /*                _
     _ __   ___  __ _| |
    | `__| / __|/ _` | |
    | |    \__ \ (_| | |
    |_|    |___/\__, |_|
                   |_|
    */

    %utlfkil(d:/xpt/want_r.xpt);

    %utl_rbegin;
    parmcards4;
      library(haven);
      library(sqldf);
      library(SASxport);
      have<-read_sas("d:/sd1/have.sas7bdat");
      have;
      want_r<-sqldf('
         select
             INDIC
            ,max(p1) as p1
            ,max(p2) as p2
            ,max(p3) as p3
            ,max(p4) as p4
            ,max(n1) as n1
         from
            (select
               INDIC
              ,case when (nam="P1") then "1" else "0" end as P1
              ,case when (nam="P2") then "1" else "0" end as P2
              ,case when (nam="P3") then "1" else "0" end as P3
              ,case when (nam="P4") then "1" else "0" end as P4
              ,case when (nam="N1") then "1" else "0" end as N1
             from
               have
            )
         group
           by INDIC
      ');
    want_r;
    write.xport(want_r,file="d:/xpt/want_r.xpt");
    ;;;;
    %utl_rend;
    run;quit;

    libname xpt xport "d:/xpt/want_r.xpt";
    proc print data=xpt.want_r;
    run;quit;

    /*           _   _                             _
     _ __  _   _| |_| |__   ___  _ __    ___  __ _| |
    | `_ \| | | | __| `_ \ / _ \| `_ \  / __|/ _` | |
    | |_) | |_| | |_| | | | (_) | | | | \__ \ (_| | |
    | .__/ \__, |\__|_| |_|\___/|_| |_| |___/\__, |_|
    |_|    |___/                                |_|
    */

    %utlfkil(d:/xpt/want_py.xpt);

    %utl_pybegin39;
    parmcards4;
    from os import path
    import pandas as pd
    import xport
    import xport.v56
    import pyreadstat
    import numpy as np
    import pandas as pd
    from pandasql import sqldf
    mysql = lambda q: sqldf(q, globals())
    from pandasql import PandaSQL
    pdsql = PandaSQL(persist=True)
    sqlite3conn = next(pdsql.conn.gen).connection.connection
    sqlite3conn.enable_load_extension(True)
    sqlite3conn.load_extension('c:/temp/libsqlitefunctions.dll')
    mysql = lambda q: sqldf(q, globals())
    have, meta = pyreadstat.read_sas7bdat("d:/sd1/have.sas7bdat")
    print(have);
    res = pdsql("""
         select
             INDIC
            ,max(p1) as p1
            ,max(p2) as p2
            ,max(p3) as p3
            ,max(p4) as p4
            ,max(n1) as n1
         from
            (select
               INDIC
              ,case when (nam='P1') then '1' else '0' end as P1
              ,case when (nam='P2') then '1' else '0' end as P2
              ,case when (nam='P3') then '1' else '0' end as P3
              ,case when (nam='P4') then '1' else '0' end as P4
              ,case when (nam='N1') then '1' else '0' end as N1
             from
               have
            )
         group
           by INDIC
        ;""")
    print(res);
    ds = xport.Dataset(res, name='want_py')
    with open('d:/xpt/want_py.xpt', 'wb') as f:
        xport.v56.dump(ds, f)
    ;;;;
    %utl_pyend39;

    libname pyxpt xport "d:/xpt/want_py.xpt";

    proc contents data=pyxpt._all_;
    run;quit;

    proc print data=pyxpt.want_py;
    run;quit;

    /*              _
      ___ _ __   __| |
     / _ \ `_ \ / _` |
    |  __/ | | | (_| |
     \___|_| |_|\__,_|

    */

    /*
     _ __ ___   __ _  ___ _ __ ___  ___
    | `_ ` _ \ / _` |/ __| `__/ _ \/ __|
    | | | | | | (_| | (__| | | (_) \__ \
    |_| |_| |_|\__,_|\___|_|  \___/|___/

    */


    %macro utl_pybegin39;
    %utlfkil(c:/temp/py_pgm.py);
    %utlfkil(c:/temp/py_pgm.log);
    filename ft15f001 "c:/temp/py_pgm.py";
    %mend utl_pybegin39;


    %macro utl_pyend39;
    run;quit;
    * EXECUTE THE PYTHON PROGRAM;
    options noxwait noxsync;
    filename rut pipe  "c:\Python39\python.exe c:/temp/py_pgm.py 2> c:/temp/py_pgm.log";
    run;quit;
    data _null_;
      file print;
      infile rut;
      input;
      put _infile_;
      putlog _infile_;
    run;quit;
    data _null_;
      infile " c:/temp/py_pgm.log";
      input;
      putlog _infile_;
    run;quit;
    %mend utl_pyend39;


    %macro utl_rbegin;
    %utlfkil(c:/temp/r_pgm.r);
    %utlfkil(c:/temp/r_pgm.log);
    filename ft15f001 "c:/temp/r_pgm.r";
    %mend utl_rbegin;



    %macro utl_rend(returnvar=N);
    * EXECUTE THE R PROGRAM;
    options noxwait noxsync;
    filename rut pipe "D:\r412\R\R-4.1.2\bin\R.exe --vanilla --quiet --no-save < c:/temp/r_pgm.r 2> c:/temp/r_pgm.log";
    run;quit;
      data _null_;
        file print;
        infile rut recfm=v lrecl=32756;
        input;
        put _infile_;
        putlog _infile_;
      run;
      filename ft15f001 clear;
      * use the clipboard to create macro variable;
      %if %upcase(%substr(&returnVar.,1,1)) ne N %then %do;
        filename clp clipbrd ;
        data _null_;
         length txt $200;
         infile clp;
         input;
         putlog "macro variable &returnVar = " _infile_;
         call symputx("&returnVar.",_infile_,"G");
        run;quit;
      %end;
    data _null_;
      file print;
      infile rut;
      input;
      put _infile_;
      putlog _infile_;
    run;quit;
    data _null_;
      infile "c:/temp/r_pgm.log";
      input;
      putlog _infile_;
    run;quit;
    %mend utl_rend;


    %macro utl_submit_r64(
          pgmx
         ,return=N
         )/des="Semi colon separated set of R commands - drop down to R";
      * write the program to a temporary file;
      filename r_pgm "%sysfunc(pathname(work))/r_pgm.txt" lrecl=32766 recfm=v;
      data _null_;
        length pgm $32756;
        file r_pgm;
        pgm=resolve(&pgmx);
        put pgm;
        putlog pgm;
      run;
      %let __loc=%sysfunc(pathname(r_pgm));
      * pipe file through R;
      filename rut pipe "D:\r412\R\R-4.1.2\bin\R.exe --vanilla --quiet --no-save < &__loc";
      data _null_;
        file print;
        infile rut recfm=v lrecl=32756;
        input;
        put _infile_;
        putlog _infile_;
      run;
      filename rut clear;
      filename r_pgm clear;

      * use the clipboard to create macro variable;
      %if %upcase(%substr(&return.,1,1)) ne N %then %do;
        filename clp clipbrd ;
        data _null_;
         length txt $200;
         infile clp;
         input;
         putlog "macro variable &return = " _infile_;
         call symputx("&return.",_infile_,"G");
        run;quit;
      %end;

    %mend utl_submit_r64;
