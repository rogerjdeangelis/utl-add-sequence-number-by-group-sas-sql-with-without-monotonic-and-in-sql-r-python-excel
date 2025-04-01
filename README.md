# utl-add-sequence-number-by-group-sas-sql-with-without-monotonic-and-in-sql-r-python-excel
Add sequence number by group sas sql with without monotonic and in sql r python excel 
    %let pgm=utl-add-sequence-number-by-group-sas-sql-with-without-monotonic-and-in-sql-r-python-excel ;

    %stop_submission;

    Add sequence number by group sas sql with without monotonic and in sql r python excel

    gitthub
    https://tinyurl.com/mt56mzut
    https://github.com/rogerjdeangelis/utl-add-sequence-number-by-group-sas-sql-with-without-monotonic-and-in-sql-r-python-excel

    communities.sas
    https://communities.sas.com/t5/SAS-Programming/Assign-sequencenumber/m-p/841124#M332572

         CONTENTS

            1 sas without monotonic (academic)
              (solution for two groups & nameXweek must be a primary key)
              (there are work arounds- nested sql..)
              not for production?

            2 sql sql with monotonic
              monotonic is used inside sqlpartition macro
              more flexible.
                Does not require sorted data
                Does not require a nameXweek primary key

            3 sql r python excel select clause
              for python and excel see
              see
              https://tinyurl.com/4e6yaap8


            4 sq lr python excel from clause

              you can apply expressiobs using suseq in the select clasue)
              for python and excel see
              see
              https://tinyurl.com/4e6yaap8


         SOAPBOX ON

           Sequence numbers and primary keys are part of many sql databases
           Sequence numbers allow functions like these i sql

           (WINDOW SQLLITE FUNCTIONS)

           RANK()
           DENSE_RANK()
           PERCENT_RANK()
           CUME_DIST()
           NTILE(N)
           LAG(expr)
           LEAD(expr)
           FIRST_VALUE(expr)
           LAST_VALUE(expr)
           NTH_VALUE(expr, N)
           PARTITION OVER

         SOAPBOX OFF

    RELATED REPOS

    https://github.com/rogerjdeangelis/utl-add-sequence-numbers-to-a-sas-table-with-and-without-sas-monotonic
    https://github.com/rogerjdeangelis/utl-adding-sequence-numbers-and-partitions-in-SAS-sql-without-using-monotonic
    https://github.com/rogerjdeangelis/utl-lags-in-proc-sql-monotonic-datastep-is-preferred
    https://github.com/rogerjdeangelis/utl-sas-keep-only-monotonic-increasing-sequences-by-group

    /*               _     _
     _ __  _ __ ___ | |__ | | ___ _ __ ___
    | `_ \| `__/ _ \| `_ \| |/ _ \ `_ ` _ \
    | |_) | | | (_) | |_) | |  __/ | | | | |
    | .__/|_|  \___/|_.__/|_|\___|_| |_| |_|
    |_|
    */


    /**************************************************************************************************************************/
    /*             INPUT           |         PROCESS                        |          OUTPUT                                 */
    /*             =====           |         =======                        |          ======                                 */
    /*                             |                                        |                                                 */
    /* NAME     WEEK        DOSE   | 1 SAS WITHOUT MONOTONIC                | ADDED VARS                                      */
    /*                             | =======================                | ==========                                      */
    /* ROGER    BASELINE    10mg   |                                        | REC SUBSEQ NAME  WEEK      DOSE                 */
    /* ROGER    WEEK1       12mg   | %let wek=0;                            |                                                 */
    /* ROGER    EOT         13mg   | %let seq=0;                            |  1    1    ROGER BASELINE  10mg                 */
    /*                             | proc sql;                              |  2    2    ROGER WEEK1     12mg                 */
    /* JANET    BASELINE    23mg   |  create                                |  3    3    ROGER EOT       13mg                 */
    /* JANET    WEEK1       21mg   |    table want(drop=_t:) as             |                                                 */
    /* JANET    WEEK2       24mg   |  select                                |  4    1    JANET BASELINE  23mg                 */
    /* JANET    EOT         23mg   |     resolve('%let seq=%eval(&seq+1);') |  5    2    JANET WEEK1     21mg                 */
    /*                             |    ,symget('seq') as rec               |  6    3    JANET WEEK2     24mg                 */
    /* options validvarname=upcase;|                                        |  7    4    JANET EOT       23mg                 */
    /* libname sd1 "d:/sd1";       |    ,resolve('%let wek=%eval(&wek+1);') |                                                 */
    /* data sd1.have;              |    ,case                               |                                                 */
    /*  input NAME$ WEEK$  DOSE$;  |      when (WEEK='BASELINE')            |                                                 */
    /* cards4;                     |         then resolve('%let wek=1;')    |                                                 */
    /* ROGER BASELINE 10mg         |      else resolve('%let wek=&wek;')    |                                                 */
    /* ROGER WEEK1    12mg         |     end                                |                                                 */
    /* ROGER EOT      13mg         |    ,symget('wek') as subSeq            |                                                 */
    /* JANET BASELINE 23mg         |                                        |                                                 */
    /* JANET WEEK1    21mg         |    ,name                               |                                                 */
    /* JANET WEEK2    24mg         |    ,week                               |                                                 */
    /* JANET EOT      23mg         |    ,dose                               |                                                 */
    /* ;;;;                        |  from                                  |                                                 */
    /* Run;quit;                   |     sd1.have                           |                                                 */
    /*                             | ;quit;                                 |                                                 */
    /*                             |                                        |                                                 */
    /*                             |------------------------------------------------------------------------------------------*/
    /*                             |                                        |                                                 */
    /*                             | 2 SQL SQL WITH MONOTONIC               | NAME     WEEK        DOSE    SUBSEQ             */
    /*                             | ========================               |                                                 */
    /*                             |                                        | JANET    WEEK2       24mg       1               */
    /*                             | proc sql;                              | JANET    WEEK1       21mg       2               */
    /*                             |   create                               | JANET    EOT         23mg       3               */
    /*                             |      table want as                     | JANET    BASELINE    23mg       4               */
    /*                             |   select                               |                                                 */
    /*                             |      name                              | ROGER    EOT         13mg       1               */
    /*                             |     ,week                              | ROGER    BASELINE    10mg       2               */
    /*                             |     ,dose                              | ROGER    WEEK1       12mg       3               */
    /*                             |     ,partition as subseq               |                                                 */
    /*                             |   from                                 |                                                 */
    /*                             |     %sqlpartition(                     |                                                 */
    /*                             |       sd1.have                         |                                                 */
    /*                             |      ,by=name)                         |                                                 */
    /*                             | ;quit;                                 |                                                 */
    /*                             |                                        |                                                 */
    /*                             |------------------------------------------------------------------------------------------*/
    /*                             |                                        |                                                 */
    /*                             | 3 SQL R PYTHON EXCEL SELECT CLAUSE     | R                                               */
    /*                             | ==================================     |                                                 */
    /*                             |                                        |    NAME     WEEK DOSE SUBSEQ                    */
    /*                             | for python and excel see               |                                                 */
    /*                             | https://tinyurl.com/4e6yaap8           | 1 JANET BASELINE 23mg      1                    */
    /*                             |                                        | 2 JANET    WEEK1 21mg      2                    */
    /*                             | %utl_rbeginx;                          | 3 JANET    WEEK2 24mg      3                    */
    /*                             | parmcards4;                            | 4 JANET      EOT 23mg      4                    */
    /*                             | library(haven)                         |                                                 */
    /*                             | library(sqldf)                         | 5 ROGER BASELINE 10mg      1                    */
    /*                             | source("c:/oto/fn_tosas9x.R")          | 6 ROGER    WEEK1 12mg      2                    */
    /*                             | have<-read_sas("d:/sd1/have.sas7bdat") | 7 ROGER      EOT 13mg      3                    */
    /*                             | want<-sqldf('                          |                                                 */
    /*                             |   select                               |                                                 */
    /*                             |      name                              | SAS                                             */
    /*                             |     ,week                              | ROWNAMES NAME  WEEK     DOSE  SUBSEQ            */
    /*                             |     ,dose                              |                                                 */
    /*                             |     ,ROW_NUMBER() OVER (               |     1    JANET BASELINE 23mg     1              */
    /*                             |         PARTITION BY name              |     2    JANET WEEK1    21mg     2              */
    /*                             |         ORDER BY  name                 |     3    JANET WEEK2    24mg     3              */
    /*                             |     ) AS Subseq                        |     4    JANET EOT      23mg     4              */
    /*                             |   from                                 |     5    ROGER BASELINE 10mg     1              */
    /*                             |      have                              |     6    ROGER WEEK1    12mg     2              */
    /*                             |   ');                                  |     7    ROGER EOT      13mg     3              */
    /*                             | print(want)                            |                                                 */
    /*                             | fn_tosas9x(                            |                                                 */
    /*                             |       inp    = want                    |                                                 */
    /*                             |      ,outlib ="d:/sd1/"                |                                                 */
    /*                             |      ,outdsn ="want"                   |                                                 */
    /*                             |      )                                 |                                                 */
    /*                             | ;;;;                                   |                                                 */
    /*                             | %utl_rendx;                            |                                                 */
    /*                             |                                        |                                                 */
    /*                             |------------------------------------------------------------------------------------------*/
    /*                             |                                        |                                                 */
    /*                             | 4 SQ R PYTHON EXCEL FROM CLAUSE        | R                                               */
    /*                             | ===============================        |    NAME     WEEK DOSE SUBSEQ                    */
    /*                             |                                        |                                                 */
    /*                             | %utl_rbeginx;                          | 1 JANET BASELINE 23mg      1                    */
    /*                             | parmcards4;                            | 2 JANET    WEEK1 21mg      2                    */
    /*                             | library(haven)                         | 3 JANET    WEEK2 24mg      3                    */
    /*                             | library(sqldf)                         | 4 JANET      EOT 23mg      4                    */
    /*                             | source("c:/oto/fn_tosas9x.R")          |                                                 */
    /*                             | have<-read_sas("d:/sd1/have.sas7bdat") | 5 ROGER BASELINE 10mg      1                    */
    /*                             | want<-sqldf('                          | 6 ROGER    WEEK1 12mg      2                    */
    /*                             |   select                               | 7 ROGER      EOT 13mg      3                    */
    /*                             |      name                              |                                                 */
    /*                             |     ,week                              |                                                 */
    /*                             |     ,dose                              | SAS                                             */
    /*                             |     ,subseq                            | ROWNAMES NAME  WEEK     DOSE  SUBSEQ            */
    /*                             |   from                                 |                                                 */
    /*                             |     (select                            |     1    JANET BASELINE 23mg     1              */
    /*                             |        name                            |     2    JANET WEEK1    21mg     2              */
    /*                             |       ,week                            |     3    JANET WEEK2    24mg     3              */
    /*                             |       ,dose                            |     4    JANET EOT      23mg     4              */
    /*                             |       ,ROW_NUMBER() OVER (             |     5    ROGER BASELINE 10mg     1              */
    /*                             |         PARTITION BY name              |     6    ROGER WEEK1    12mg     2              */
    /*                             |         ORDER BY  name ) as subseq     |     7    ROGER EOT      13mg     3              */
    /*                             |     from                               |                                                 */
    /*                             |        have )                          |                                                 */
    /*                             |     ')                                 |                                                 */
    /*                             | print(want)                            |                                                 */
    /*                             | fn_tosas9x(                            |                                                 */
    /*                             |       inp    = want                    |                                                 */
    /*                             |      ,outlib ="d:/sd1/"                |                                                 */
    /*                             |      ,outdsn ="want"                   |                                                 */
    /*                             |      )                                 |                                                 */
    /*                             | ;;;;                                   |                                                 */
    /*                             | %utl_rendx;                            |                                                 */
    /**************************************************************************************************************************/

    /*                   _
    (_)_ __  _ __  _   _| |_
    | | `_ \| `_ \| | | | __|
    | | | | | |_) | |_| | |_
    |_|_| |_| .__/ \__,_|\__|
            |_|
    */

    options validvarname=upcase;
    libname sd1 "d:/sd1";
    data sd1.have;
     input NAME$ WEEK$  DOSE$;
    cards4;
    ROGER BASELINE 10mg
    ROGER WEEK1    12mg
    ROGER EOT      13mg
    JANET BASELINE 23mg
    JANET WEEK1    21mg
    JANET WEEK2    24mg
    JANET EOT      23mg
    ;;;;
    Run;quit;

    /**************************************************************************************************************************/
    /* NAME     WEEK        DOSE                                                                                              */
    /*                                                                                                                        */
    /* ROGER    BASELINE    10mg                                                                                              */
    /* ROGER    WEEK1       12mg                                                                                              */
    /* ROGER    EOT         13mg                                                                                              */
    /*                                                                                                                        */
    /* JANET    BASELINE    23mg                                                                                              */
    /* JANET    WEEK1       21mg                                                                                              */
    /* JANET    WEEK2       24mg                                                                                              */
    /* JANET    EOT         23mg                                                                                              */
    /**************************************************************************************************************************/

    /*                            _ _   _                 _                                _              _
    / |  ___  __ _ ___  __      _(_) |_| |__   ___  _   _| |_  _ __ ___   ___  _ __   ___ | |_ ___  _ __ (_) ___
    | | / __|/ _` / __| \ \ /\ / / | __| `_ \ / _ \| | | | __|| `_ ` _ \ / _ \| `_ \ / _ \| __/ _ \| `_ \| |/ __|
    | | \__ \ (_| \__ \  \ V  V /| | |_| | | | (_) | |_| | |_ | | | | | | (_) | | | | (_) | || (_) | | | | | (__
    |_| |___/\__,_|___/   \_/\_/ |_|\__|_| |_|\___/ \__,_|\__||_| |_| |_|\___/|_| |_|\___/ \__\___/|_| |_|_|\___|
    */

    %let wek=0;
    %let seq=0;
    proc sql;
     create
       table want(drop=_t:) as
     select
        resolve('%let seq=%eval(&seq+1);')
       ,symget('seq') as rec

       ,resolve('%let wek=%eval(&wek+1);')
       ,case
         when (WEEK='BASELINE')
            then resolve('%let wek=1;')
         else resolve('%let wek=&wek;')
        end
       ,symget('wek') as subSeq

       ,name
       ,week
       ,dose
     from
        sd1.have
    ;quit;

    /**************************************************************************************************************************/
    /*  REC    SUBSEQ    NAME     WEEK        DOSE                                                                            */
    /*                                                                                                                        */
    /*  1       1       ROGER    BASELINE    10mg                                                                             */
    /*  2       2       ROGER    WEEK1       12mg                                                                             */
    /*  3       3       ROGER    EOT         13mg                                                                             */
    /*  4       1       JANET    BASELINE    23mg                                                                             */
    /*  5       2       JANET    WEEK1       21mg                                                                             */
    /*  6       3       JANET    WEEK2       24mg                                                                             */
    /*  7       4       JANET    EOT         23mg                                                                             */
    /**************************************************************************************************************************/

    /*___              _             _            _ _   _                                  _              _
    |___ \   ___  __ _| |  ___  __ _| | __      _(_) |_| |__   _ __ ___   ___  _ __   ___ | |_ ___  _ __ (_) ___
      __) | / __|/ _` | | / __|/ _` | | \ \ /\ / / | __| `_ \ | `_ ` _ \ / _ \| `_ \ / _ \| __/ _ \| `_ \| |/ __|
     / __/  \__ \ (_| | | \__ \ (_| | |  \ V  V /| | |_| | | || | | | | | (_) | | | | (_) | || (_) | | | | | (__
    |_____| |___/\__, |_| |___/\__, |_|   \_/\_/ |_|\__|_| |_||_| |_| |_|\___/|_| |_|\___/ \__\___/|_| |_|_|\___|
                    |_|           |_|
    */

    proc sql;
      create
         table want as
      select
         name
        ,week
        ,dose
        ,partition as subseq
      from
        %sqlpartition(
          sd1.have
         ,by=name)
    ;quit;

    /**************************************************************************************************************************/
    /* NAME     WEEK        DOSE    SUBSEQ                                                                                    */
    /*                                                                                                                        */
    /* JANET    WEEK2       24mg       1                                                                                      */
    /* JANET    WEEK1       21mg       2                                                                                      */
    /* JANET    EOT         23mg       3                                                                                      */
    /* JANET    BASELINE    23mg       4                                                                                      */
    /* ROGER    EOT         13mg       1                                                                                      */
    /* ROGER    BASELINE    10mg       2                                                                                      */
    /* ROGER    WEEK1       12mg       3                                                                                      */
    /**************************************************************************************************************************/

    /*____             _                      _   _                                    _            _           _          _
    |___ /   ___  __ _| |  _ __   _ __  _   _| |_| |__   ___  _ __    _____  _____ ___| |  ___  ___| | ___  ___| |_    ___| | __ _ _   _ ___  ___
      |_ \  / __|/ _` | | | `__| | `_ \| | | | __| `_ \ / _ \| `_ \  / _ \ \/ / __/ _ \ | / __|/ _ \ |/ _ \/ __| __|  / __| |/ _` | | | / __|/ _ \
     ___) | \__ \ (_| | | | |    | |_) | |_| | |_| | | | (_) | | | ||  __/>  < (_|  __/ | \__ \  __/ |  __/ (__| |_  | (__| | (_| | |_| \__ \  __/
    |____/  |___/\__, |_| |_|    | .__/ \__, |\__|_| |_|\___/|_| |_| \___/_/\_\___\___|_| |___/\___|_|\___|\___|\__|  \___|_|\__,_|\__,_|___/\___|
                    |_|          |_|    |___/

    */

    /*---- for python and excel see      ----*/
    /*---- https://tinyurl.com/4e6yaap8  ----*/

    %utl_rbeginx;
    parmcards4;
    library(haven)
    library(sqldf)
    source("c:/oto/fn_tosas9x.R")
    have<-read_sas("d:/sd1/have.sas7bdat")
    want<-sqldf('
      select
         name
        ,week
        ,dose
        ,ROW_NUMBER() OVER (
            PARTITION BY name
            ORDER BY  name
        ) AS Subseq
      from
         have
      ');
    print(want)
    fn_tosas9x(
          inp    = want
         ,outlib ="d:/sd1/"
         ,outdsn ="want"
         )
    ;;;;
    %utl_rendx;

    /**************************************************************************************************************************/
    /* R                            | SAS                                                                                     */
    /*  name     week dose subseq   | ROWNAMES    NAME     WEEK        DOSE    SUBSEQ                                         */
    /*                              |                                                                                         */
    /* JANET BASELINE 23mg      1   |     1       JANET    BASELINE    23mg       1                                           */
    /* JANET    WEEK1 21mg      2   |     2       JANET    WEEK1       21mg       2                                           */
    /* JANET    WEEK2 24mg      3   |     3       JANET    WEEK2       24mg       3                                           */
    /* JANET      EOT 23mg      4   |     4       JANET    EOT         23mg       4                                           */
    /* ROGER BASELINE 10mg      1   |     5       ROGER    BASELINE    10mg       1                                           */
    /* ROGER    WEEK1 12mg      2   |     6       ROGER    WEEK1       12mg       2                                           */
    /* ROGER      EOT 13mg      3   |     7       ROGER    EOT         13mg       3                                           */
    /***************************************************************************************************************************/

    /*  _                 _                    _   _                                    _    __                            _
    | || |    ___  __ _  | |_ __   _ __  _   _| |_| |__   ___  _ __    _____  _____ ___| |  / _|_ __ ___  _ __ ___     ___| | __ _ _   _ ___  ___
    | || |_  / __|/ _` | | | `__| | `_ \| | | | __| `_ \ / _ \| `_ \  / _ \ \/ / __/ _ \ | | |_| `__/ _ \| `_ ` _ \   / __| |/ _` | | | / __|/ _ \
    |__   _| \__ \ (_| | | | |    | |_) | |_| | |_| | | | (_) | | | ||  __/>  < (_|  __/ | |  _| | | (_) | | | | | | | (__| | (_| | |_| \__ \  __/
       |_|   |___/\__, | |_|_|    | .__/ \__, |\__|_| |_|\___/|_| |_| \___/_/\_\___\___|_| |_| |_|  \___/|_| |_| |_|  \___|_|\__,_|\__,_|___/\___|
                     |_|          |_|    |___/
    */

    %utl_rbeginx;
    parmcards4;
    library(haven)
    library(sqldf)
    source("c:/oto/fn_tosas9x.R")
    have<-read_sas("d:/sd1/have.sas7bdat")
    want<-sqldf('
      select
         name
        ,week
        ,dose
        ,subseq
      from
        (select
           name
          ,week
          ,dose
          ,ROW_NUMBER() OVER (
            PARTITION BY name
            ORDER BY  name ) as subseq
        from
           have )
        ')
    print(want)
    fn_tosas9x(
          inp    = want
         ,outlib ="d:/sd1/"
         ,outdsn ="want"
         )
    ;;;;
    %utl_rendx;

    /**************************************************************************************************************************/
    /* R                            | SAS                                                                                     */
    /*  name     week dose subseq   | ROWNAMES    NAME     WEEK        DOSE    SUBSEQ                                         */
    /*                              |                                                                                         */
    /* JANET BASELINE 23mg      1   |     1       JANET    BASELINE    23mg       1                                           */
    /* JANET    WEEK1 21mg      2   |     2       JANET    WEEK1       21mg       2                                           */
    /* JANET    WEEK2 24mg      3   |     3       JANET    WEEK2       24mg       3                                           */
    /* JANET      EOT 23mg      4   |     4       JANET    EOT         23mg       4                                           */
    /* ROGER BASELINE 10mg      1   |     5       ROGER    BASELINE    10mg       1                                           */
    /* ROGER    WEEK1 12mg      2   |     6       ROGER    WEEK1       12mg       2                                           */
    /* ROGER      EOT 13mg      3   |     7       ROGER    EOT         13mg       3                                           */
    /***************************************************************************************************************************/

    /*              _
      ___ _ __   __| |
     / _ \ `_ \ / _` |
    |  __/ | | | (_| |
     \___|_| |_|\__,_|

    */
