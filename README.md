# utl_matching_treatment_to_mutiple_control_records_populating_first_episode_of_dry_mouth
Matching treatment to mutiple control records populating first episode of dry mouth. Keywords: sas sql join merge big data analytics macros oracle teradata mysql sas communities stackoverflow statistics artificial inteligence AI Python R Java Javascript WPS Matlab SPSS Scala Perl C C# Excel MS Access JSON graphics maps NLP natural language processing machine learning igraph DOSUBL DOW loop stackoverfl SAS community.
    Matching treatment to mutiple control records populating first episode of dry mouth

    github
    https://goo.gl/Si8hy6
    https://github.com/rogerjdeangelis/utl_matching_treatment_to_mutiple_control_records_populating_first_episode_of_dry_mouth

    see
    https://goo.gl/gJhooQ
    https://communities.sas.com/t5/Base-SAS-Programming/matching-without-replacement/m-p/433019

    KSharp profile
    https://communities.sas.com/t5/user/viewprofilepage/user-id/18408


    INPUT
    =====

    Same results in SAS and WPS

    ALGORITHM

       1. Load the control table into a hash table(H)
          Key FYEAR SIC and data CTRLID
       2. Create a skinny second hash table(K)
          with just the unique CTRLIDs from cntrol table
       3. Cycle through the treatment table
          If match with control hash table(H) then
          check CTRLID in skinny K hash table to see if we have used CTRLID
          If not used that output and move to the next record in treatment

    JOIN ON FYEAR SIC

    WORK.TREATMENT total obs=7

        FYEAR    SIC   TRTID
        1998     73    21878  *  first match on fyear and sic in control table.
                                 Output this record with ctrlid=11735 (dry month)
                                 for future matches where ctrlid=11735 set ctrlid=blank(dry mouth)
        1999     73    11809
        2005     73     8598  *  first match in control same process as above with 11088
        2005     73    11901
        2007     73    41671
        2008     73     6278
        2008     73    16167

    WORK.CONTROL total obs=21    |
                                 |
       FYEAR    SIC   CTRLID     |
        1995     73    11735     |
        1996     73    11735     |
        1997     73    11735     |
        1998     73    11735     | *    *** This is the matched record with treatment (1998 73)
                                        *** 11735 will not appear again in the output
        1999     73    11735     |
        2000     73    11735     |
        2001     73    11735     |
        2002     73    11088     |
        2002     73    11735     |
        2003     73    11088     |
        2003     73    11735     |
        2004     73    11735     |
        2005     73    11088     | *    *** This is the matched record with treatment (2005 73)
        2005     73    11735     |      *** 11088 will not appear again in the output
        2006     73    11088     |
        2006     73    11735     |
        2007     73    11088     |
        2007     73    11735     |
        2008     73    11088     |
        2008     73    11735     |
        2009     73    11088     |


    PROCESS (All the code)
    ======================

       data want;
        if _n_ eq 1 then do;
         if 0 then set control;
         declare hash h(dataset:'control',multidata:'y');
         h.definekey('fyear','sic');
         h.definedata('ctrlid');
         h.definedone();
         declare hash k();
         k.definekey('ctrlid');
         k.definedone();
        end;
       set treatment;
       put _all_;
       call missing(ctrlid);
       rc=h.find();
           do while(rc=0);
             if k.check()=0 then call missing(ctrlid);
             else do;k.add();leave;end;
             rc=h.find_next();
           end;
       drop rc;
       run;

    OUTPUT
    ======

     WORK.WANT total obs=7

       FYEAR    SIC    CTRLID    TRTID

        1998     73     11735    21878
        1999     73         .    11809
        2005     73     11088     8598
        2005     73         .    11901
        2007     73         .    41671
        2008     73         .     6278
        2008     73         .    16167

    *                _              _       _
     _ __ ___   __ _| | _____    __| | __ _| |_ __ _
    | '_ ` _ \ / _` | |/ / _ \  / _` |/ _` | __/ _` |
    | | | | | | (_| |   <  __/ | (_| | (_| | || (_| |
    |_| |_| |_|\__,_|_|\_\___|  \__,_|\__,_|\__\__,_|

    ;

    data treatment;
    retain fyear sic trtid;
    input trtid fyear sic;
    cards4;
    6278 2008  73
    8598 2005  73
    11809 1999 73
    11901 2005 73
    16167 2008 73
    21878 1998 73
    41671 2007 73
    ;;;;
    run;quit;

    proc sort data=treatment;
    by fyear sic;
    run;quit;

    data control;
    retain fyear sic ;
    input ctrlid fyear sic;
    cards;
    11088 2002 73 1
    11088 2003 73 2
    11088 2005 73 3
    11088 2006 73 4
    11088 2007 73 5
    11088 2008 73 6
    11088 2009 73 7
    11735 1995 73 8
    11735 1996 73 9
    11735 1997 73 10
    11735 1998 73 11
    11735 1999 73 12
    11735 2000 73 13
    11735 2001 73 14
    11735 2002 73 15
    11735 2003 73 16
    11735 2004 73 17
    11735 2005 73 18
    11735 2006 73 19
    11735 2007 73 20
    11735 2008 73 21
    ;;;;
    run;quit;

    proc sort data=control;
    by fyear sic ;
    run;quit;


    *          _       _   _
     ___  ___ | |_   _| |_(_) ___  _ __
    / __|/ _ \| | | | | __| |/ _ \| '_ \
    \__ \ (_) | | |_| | |_| | (_) | | | |
    |___/\___/|_|\__,_|\__|_|\___/|_| |_|

    ;

    %utl_submit_wps64('
    libname wrk sas7bdat "%sysfunc(pathname(work))";
    data wrk.matched;
     if _n_ eq 1 then do;
      if 0 then set wrk.control;
      declare hash h(dataset:"wrk.control",multidata:"y");
      h.definekey("fyear","sic");
      h.definedata("ctrlid");
      h.definedone();

      declare hash k();
      k.definekey("ctrlid");
      k.definedone();
     end;
    set wrk.treatment;
    call missing(ctrlid);
    rc=h.find();
        do while(rc=0);
          if k.check()=0 then call missing(ctrlid);
            else do;k.add();leave;end;
          rc=h.find_next();
        end;
    drop rc;
    run;
    run;quit;
    ');

