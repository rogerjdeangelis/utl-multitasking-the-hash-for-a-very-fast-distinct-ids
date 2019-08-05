# utl-multitasking-the-hash-for-a-very-fast-distinct-ids
Provide fast algorithms for sql select distinct pid from have ( 1 billion observations 50gb )
    Problem: Provide fast algorithms for sql select distinct pid from have ( 1 billion observations 50gb )                    
                                                                                                                              
     BENCHMARKS                                                                                                               
                                                                                                                              
        MINUTES                                                                                                               
        -------                                                                                                               
                                                                                                                              
           7.5  Single SQL                                                                                                    
                                                                                                                              
           3.5  Multitask SQL (set CPUCOUNT=1 and THREADS=1 with 8 paralell tasks)                                            
                                                                                                                              
                Output is a view of 8 concatenated data SAS tables.                                                           
                A view is preferred, it as almost as fast as reading a physical dataset. Eight partitioned tables             
                can be very useful. (you can decide to keep or manipulate other variables)                                    
                (add 1 minute more if you want a physical dataset.                                                            
                                                                                                                              
            5   Proc sort nodupkey by pid                                                                                     
                                                                                                                              
            4   Multitask Sort  (output is a view composed of 8 concatenated tables)                                          
                                                                                                                              
          2.5   Multitask HASH                                                                                                
                                                                                                                              
                Output is a view of * concatenated data SAS tables.                                                           
                A view is preferred, it as almost as fast as reading a physical dataset. Eight partitioned tables             
                can be very useful. DOES NOT MATTER THAT THE HASH IS SINGLE THREADED IF YOU HAVE EIGHT CORES.                 
                (add 1 minute more if you want a physical dataset. More flexible than SQL or a Sort.                          
                Make sure none of the 8 parallel tasks require more than 2gb of memory.                                       
                                                                                                                              
    *_                   _                                                                                                    
    (_)_ __  _ __  _   _| |_ ___                                                                                              
    | | '_ \| '_ \| | | | __/ __|                                                                                             
    | | | | | |_) | |_| | |_\__ \                                                                                             
    |_|_| |_| .__/ \__,_|\__|___/                                                                                             
            |_|                                                                                                               
                      _   _ _   _                      _                                                                      
     _ __   __ _ _ __| |_(_) |_(_) ___  _ __   ___  __| |                                                                     
    | '_ \ / _` | '__| __| | __| |/ _ \| '_ \ / _ \/ _` |                                                                     
    | |_) | (_| | |  | |_| | |_| | (_) | | | |  __/ (_| |                                                                     
    | .__/ \__,_|_|   \__|_|\__|_|\___/|_| |_|\___|\__,_|                                                                     
    |_|                                                                                                                       
    ;                                                                                                                         
                                                                                                                              
    %let monthsq="JAN","FEB","MAR","APR","MAY","JUN","JUL","AUG","SEP","OCT", "NOV", "DEC" ;                                  
    Libname unq "f:/unq";                                                                                                     
    data unq.have(bufno=1000);;                                                                                               
      retain pid 0 mth "   " line 0;                                                                                          
      array nums[5] $5 ("DGN01","DGN02","DGN03","DGN04","DGN05");                                                             
      call streaminit(1234);                                                                                                  
      do mth=&monthsq;                                                                                                        
        do i=1 to 1e7;                                                                                                        
          pid = int(1e11*rand("uniform"));                                                                                    
          do line=1 to int(15*rand("uniform"))+1;                                                                             
            output;                                                                                                           
          end;                                                                                                                
        end;                                                                                                                  
      end;                                                                                                                    
      drop i;                                                                                                                 
    run;quit;                                                                                                                 
                                                                                                                              
    /*                                                                                                                        
                                                                                                                              
    NOTE: The data set UNQ.HAVE has 959992705 observations and 8 variables.                                                   
    NOTE: DATA statement used (Total process time):                                                                           
          real time           3:11.58                                                                                         
          user cpu time       58.90 seconds                                                                                   
                                                                                                                              
    Approximately 1 billion observations                                                                                      
                                                                                                                              
    SPDE.HAVE total obs=959,992,705                                                                                           
                                                                                                                              
           PID        MTH    LINE    NUMS1    NUMS2    NUMS3    NUMS4    NUMS5                                                
                                                                                                                              
       75661130878    JAN      1     DGN01    DGN02    DGN03    DGN04    DGN05                                                
       75661130878    JAN      2     DGN01    DGN02    DGN03    DGN04    DGN05                                                
       75661130878    JAN      3     DGN01    DGN02    DGN03    DGN04    DGN05                                                
       75661130878    JAN      4     DGN01    DGN02    DGN03    DGN04    DGN05                                                
       75661130878    JAN      5     DGN01    DGN02    DGN03    DGN04    DGN05                                                
       75661130878    JAN      6     DGN01    DGN02    DGN03    DGN04    DGN05                                                
       75661130878    JAN      7     DGN01    DGN02    DGN03    DGN04    DGN05                                                
       75661130878    JAN      8     DGN01    DGN02    DGN03    DGN04    DGN05                                                
       75661130878    JAN      9     DGN01    DGN02    DGN03    DGN04    DGN05                                                
       75661130878    JAN     10     DGN01    DGN02    DGN03    DGN04    DGN05                                                
       75661130878    JAN     11     DGN01    DGN02    DGN03    DGN04    DGN05                                                
       75661130878    JAN     12     DGN01    DGN02    DGN03    DGN04    DGN05                                                
                                                                                                                              
       29754594038    JAN      1     DGN01    DGN02    DGN03    DGN04    DGN05                                                
       29754594038    JAN      2     DGN01    DGN02    DGN03    DGN04    DGN05                                                
       29754594038    JAN      3     DGN01    DGN02    DGN03    DGN04    DGN05                                                
       29754594038    JAN      4     DGN01    DGN02    DGN03    DGN04    DGN05                                                
       29754594038    JAN      5     DGN01    DGN02    DGN03    DGN04    DGN05                                                
                                                                                                                              
       99591946043    JAN      1     DGN01    DGN02    DGN03    DGN04    DGN05                                                
       99591946043    JAN      2     DGN01    DGN02    DGN03    DGN04    DGN05                                                
       99591946043    JAN      3     DGN01    DGN02    DGN03    DGN04    DGN05                                                
       99591946043    JAN      4     DGN01    DGN02    DGN03    DGN04    DGN05                                                
                                                                                                                              
       29566688230    JAN      1     DGN01    DGN02    DGN03    DGN04    DGN05                                                
    */                                                                                                                        
                                                                                                                              
                                                                                                                              
    *            _               _                                                                                            
      ___  _   _| |_ _ __  _   _| |_                                                                                          
     / _ \| | | | __| '_ \| | | | __|                                                                                         
    | (_) | |_| | |_| |_) | |_| | |_                                                                                          
     \___/ \__,_|\__| .__/ \__,_|\__|                                                                                         
                    |_|                                                                                                       
    ;                                                                                                                         
                                                                                                                              
    SOUT.DISTINCT_PID (118,339,280 dustinct ids)                                                                              
                                                                                                                              
                    DISTINCT_                                                                                                 
           Obs         PID                                                                                                    
                                                                                                                              
             1          186                                                                                                   
             2          977                                                                                                   
             3         1280                                                                                                   
             4         1909                                                                                                   
             5         2211                                                                                                   
             6         3306                                                                                                   
             7         4307                                                                                                   
             8         5541                                                                                                   
             9         6356                                                                                                   
           ...         ...                                                                                                    
                                                                                                                              
     118339280  99996688230                                                                                                   
                                                                                                                              
                                                                                                                              
    *          _       _   _                                                                                                  
     ___  ___ | |_   _| |_(_) ___  _ __  ___                                                                                  
    / __|/ _ \| | | | | __| |/ _ \| '_ \/ __|                                                                                 
    \__ \ (_) | | |_| | |_| | (_) | | | \__ \                                                                                 
    |___/\___/|_|\__,_|\__|_|\___/|_| |_|___/                                                                                 
                                                                                                                              
    ;                                                                                                                         
                                                                                                                              
    *          _   _       _                      _   _                                                                       
     ___  __ _| | (_)_ __ | |_ ___ _ __ __ _  ___| |_(_)_   _____                                                             
    / __|/ _` | | | | '_ \| __/ _ \ '__/ _` |/ __| __| \ \ / / _ \                                                            
    \__ \ (_| | | | | | | | ||  __/ | | (_| | (__| |_| |\ V /  __/                                                            
    |___/\__, |_| |_|_| |_|\__\___|_|  \__,_|\___|\__|_| \_/ \___|                                                            
            |_|                                                                                                               
    ;                                                                                                                         
                                                                                                                              
    libname unq "f:/unq";                                                                                                     
    proc sql;                                                                                                                 
      create                                                                                                                  
         table pid_Brute_Sql as                                                                                               
      select                                                                                                                  
         distinct pid as pid_rawSql                                                                                           
      from                                                                                                                    
         unq.have                                                                                                             
    ;quit;                                                                                                                    
                                                                                                                              
    NOTE: Table WORK.PID_BRUTE_SQL created, with 118339280 rows and 1 columns.                                                
                                                                                                                              
    36  !  quit;                                                                                                              
    NOTE: PROCEDURE SQL used (Total process time):                                                                            
          real time           7:35.30                                                                                         
          cpu time            24:22.76                                                                                        
                                                                                                                              
    *          _   _           _       _                                                                                      
     ___  __ _| | | |__   __ _| |_ ___| |__                                                                                   
    / __|/ _` | | | '_ \ / _` | __/ __| '_ \                                                                                  
    \__ \ (_| | | | |_) | (_| | || (__| | | |                                                                                 
    |___/\__, |_| |_.__/ \__,_|\__\___|_| |_|                                                                                 
            |_|                                                                                                               
    ;                                                                                                                         
                                                                                                                              
    filename ft15f001 "c:/oto/unq_sql.sas";                                                                                   
    parmcards4;                                                                                                               
    %macro unq_sql(drv,beg,end);                                                                                              
    options source;                                                                                                           
    libname unq "f:/unq";                                                                                                     
    libname &drv "&drv.:/wrk";                                                                                                
    options cpucount=1 threads=1;                                                                                             
    proc sql;                                                                                                                 
      create                                                                                                                  
         table &drv..sql_pid&beg(bufno=500) as                                                                                
      select                                                                                                                  
         distinct pid as distinct_pid                                                                                         
      from                                                                                                                    
         unq.have(firstobs=&beg obs=&end)                                                                                     
    ;quit;                                                                                                                    
    %mend unq_sql;                                                                                                            
    ;;;;                                                                                                                      
    run;quit;                                                                                                                 
                                                                                                                              
    *test;                                                                                                                    
    %unq_sql(c,1,5000000)                                                                                                     
                                                                                                                              
                                                                                                                              
    %let _s=%sysfunc(compbl(C:\Progra~1\SASHome\SASFoundation\9.4\sas.exe -sysin c:\nul -rsasuser                             
     -sasautos c:\oto -work f:\wrk -config c:\cfg\cfgsas94m6.cfg));                                                           
                                                                                                                              
    systask kill sys1 sys2 sys3 sys4 sys5 sys6 sys7 sys8 ;                                                                    
    systask command "&_s -log d:\unq\log\unq1.log -termstmt '%nrstr(%unq_sql(c,1,125000000))'" taskname=sys1;                 
    systask command "&_s -log d:\unq\log\unq2.log -termstmt '%nrstr(%unq_sql(d,125000001,250000000))'" taskname=sys2;         
    systask command "&_s -log d:\unq\log\unq3.log -termstmt '%nrstr(%unq_sql(e,250000001,375000000))'" taskname=sys3;         
    systask command "&_s -log d:\unq\log\unq4.log -termstmt '%nrstr(%unq_sql(f,375000001,500000000))'" taskname=sys4;         
    systask command "&_s -log d:\unq\log\unq5.log -termstmt '%nrstr(%unq_sql(i,500000001,625000000))'" taskname=sys5;         
    systask command "&_s -log d:\unq\log\unq6.log -termstmt '%nrstr(%unq_sql(c,625000001,750000000))'" taskname=sys6;         
    systask command "&_s -log d:\unq\log\unq7.log -termstmt '%nrstr(%unq_sql(e,750000001,875000000))'" taskname=sys7;         
    systask command "&_s -log d:\unq\log\unq8.log -termstmt '%nrstr(%unq_sql(f,875000001,1000000000))'" taskname=sys8;        
    waitfor sys1 sys2 sys3 sys4 sys5 sys6 sys7 sys8 ;                                                                         
                                                                                                                              
    run;quit;                                                                                                                 
                                                                                                                              
    /* longest run                                                                                                            
    NOTE: Table I.SQL_PID500000001 created, with 15595718 rows and 1 columns.                                                 
                                                                                                                              
    NOTE: PROCEDURE SQL used (Total process time):                                                                            
          real time           3:46.72                                                                                         
          cpu time            1:55.56                                                                                         
    */                                                                                                                        
                                                                                                                              
    libname  c  "c:/wrk";                                                                                                     
    libname  d  "d:/wrk";                                                                                                     
    libname  e  "e:/wrk";                                                                                                     
    libname  f  "f:/wrk";                                                                                                     
    libname  i  "i:/wrk";                                                                                                     
                                                                                                                              
    /* keep this as a view ? */                                                                                               
    data allhsh/view=allhsh;                                                                                                  
      set                                                                                                                     
        c.sql_pid1                                                                                                            
        d.sql_pid125000001                                                                                                    
        e.sql_pid250000001                                                                                                    
        f.sql_pid375000001                                                                                                    
        i.sql_pid500000001                                                                                                    
        c.sql_pid625000001                                                                                                    
        e.sql_pid750000001                                                                                                    
        f.sql_pid875000000 end=dne;                                                                                           
      by distinct_pid;                                                                                                        
      if first.distinct_pid then output;                                                                                      
    run;quit;                                                                                                                 
                                                                                                                              
    /*                                                                                                                        
    NOTE: There were 15597099 observations read from the data set C.SQL_PID1.                                                 
    NOTE: There were 15595691 observations read from the data set D.SQL_PID125000001.                                         
    NOTE: There were 15596531 observations read from the data set E.SQL_PID250000001.                                         
    NOTE: There were 15599366 observations read from the data set F.SQL_PID375000001.                                         
    NOTE: There were 15595718 observations read from the data set I.SQL_PID500000001.                                         
    NOTE: There were 15596024 observations read from the data set C.SQL_PID625000001.                                         
    NOTE: There were 15593422 observations read from the data set E.SQL_PID750000001.                                         
    NOTE: There were 10613886 observations read from the data set F.SQL_PID875000000.                                         
    500 !  quit;                                                                                                              
    NOTE: PROCEDURE SQL used (Total process time):                                                                            
          real time           58.44 seconds                                                                                   
          user cpu time       56.45 seconds                                                                                   
    */                                                                                                                        
                                                                                                                              
                                                                                                                              
    /* If you want count distinct */                                                                                          
                                                                                                                              
    proc sql;                                                                                                                 
       select count(*) format=comma23. from work.allhsh                                                                       
    ;quit;                                                                                                                    
                                                                                                                              
    /*                                                                                                                        
    NOTE: The PROCEDURE SQL printed page 1.                                                                                   
    NOTE: PROCEDURE SQL used (Total process time):                                                                            
          real time           55.19 seconds                                                                                   
          cpu time            59.60 seconds                                                                                   
                                                                                                                              
                                                                                                                              
    * same as proc sql                                                                                                        
     118,339,280                                                                                                              
    */                                                                                                                        
                                                                                                                              
                                                                                                                              
    *                _                     _                                                                                  
      ___  ___  _ __| |_   _ __   ___   __| |_   _ _ __                                                                       
     / __|/ _ \| '__| __| | '_ \ / _ \ / _` | | | | '_ \                                                                      
     \__ \ (_) | |  | |_  | | | | (_) | (_| | |_| | |_) |                                                                     
     |___/\___/|_|   \__| |_| |_|\___/ \__,_|\__,_| .__/                                                                      
                                                  |_|                                                                         
    ;                                                                                                                         
    * note utilities files are on separate drive e:/wrk and sortsize=4gb;                                                     
                                                                                                                              
    libname f "f:/unq";                                                                                                       
    libname c "c:/wrk";                                                                                                       
    options  ubufno=20;                                                                                                       
    proc sort data=f.have(keep=pid) out=c.havsrt nodupkey noequals;                                                           
      by pid;                                                                                                                 
    run;                                                                                                                      
                                                                                                                              
    /*                                                                                                                        
    NOTE: 841653425 observations with duplicate key values were deleted.                                                      
    NOTE: The data set C.HAVSRT has 118339280 observations and 1 variables.                                                   
    NOTE: PROCEDURE SORT used (Total process time):                                                                           
          real time           4:56.94                                                                                         
          cpu time            19:53.03                                                                                        
    */                                                                                                                        
                                                                                                                              
    *               _     _           _       _                                                                               
     ___  ___  _ __| |_  | |__   __ _| |_ ___| |__                                                                            
    / __|/ _ \| '__| __| | '_ \ / _` | __/ __| '_ \                                                                           
    \__ \ (_) | |  | |_  | |_) | (_| | || (__| | | |                                                                          
    |___/\___/|_|   \__| |_.__/ \__,_|\__\___|_| |_|                                                                          
                                                                                                                              
    ;                                                                                                                         
                                                                                                                              
    filename ft15f001 "c:/oto/unq_srt.sas";                                                                                   
    parmcards4;                                                                                                               
    %macro unq_srt(drv,beg,end);                                                                                              
    options source;                                                                                                           
    libname f "f:/unq";                                                                                                       
    libname drv "&drv.:/wrk";                                                                                                 
    options  ubufno=20;                                                                                                       
    proc sort data=f.have(firstobs=&beg obs=&end keep=pid) out=drv.srt&beg nodupkey noequals;                                 
      by pid;                                                                                                                 
    run;quit;                                                                                                                 
    %mend unq_srt;                                                                                                            
    ;;;;                                                                                                                      
    run;quit;                                                                                                                 
                                                                                                                              
    *test;                                                                                                                    
    %unq_srt(c,1,5000000)                                                                                                     
                                                                                                                              
    %let _s=%sysfunc(compbl(C:\Progra~1\SASHome\SASFoundation\9.4\sas.exe -sysin c:\nul -rsasuser                             
     -sasautos c:\oto -work f:\wrk -config c:\cfg\cfgsas94m6.cfg -utilloc e:/wrk));                                           
                                                                                                                              
    systask kill sys1 sys2 sys3 sys4 sys5 sys6 sys7 sys8 ;                                                                    
    systask command "&_s -log d:\unq\log\unq1.log -termstmt '%nrstr(%unq_srt(c,1,125000000))'" taskname=sys1;                 
    systask command "&_s -log d:\unq\log\unq2.log -termstmt '%nrstr(%unq_srt(d,125000001,250000000))'" taskname=sys2;         
    systask command "&_s -log d:\unq\log\unq3.log -termstmt '%nrstr(%unq_srt(e,250000001,375000000))'" taskname=sys3;         
    systask command "&_s -log d:\unq\log\unq4.log -termstmt '%nrstr(%unq_srt(f,375000001,500000000))'" taskname=sys4;         
    systask command "&_s -log d:\unq\log\unq5.log -termstmt '%nrstr(%unq_srt(i,500000001,625000000))'" taskname=sys5;         
    systask command "&_s -log d:\unq\log\unq6.log -termstmt '%nrstr(%unq_srt(c,625000001,750000000))'" taskname=sys6;         
    systask command "&_s -log d:\unq\log\unq7.log -termstmt '%nrstr(%unq_srt(e,750000001,875000000))'" taskname=sys7;         
    systask command "&_s -log d:\unq\log\unq8.log -termstmt '%nrstr(%unq_srt(f,875000001,1000000000))'" taskname=sys8;        
    waitfor sys1 sys2 sys3 sys4 sys5 sys6 sys7 sys8 ;                                                                         
                                                                                                                              
    run;quit;                                                                                                                 
                                                                                                                              
    /* longest run                                                                                                            
    NOTE: Table C.SQL_PID1 created, with 15597099 rows and 1 columns.                                                         
                                                                                                                              
    NOTE: PROCEDURE SQL used (Total process time):                                                                            
          real time           2:55.82                                                                                         
          cpu time            2:10.19                                                                                         
    */                                                                                                                        
                                                                                                                              
    libname  c  "c:/wrk";                                                                                                     
    libname  d  "d:/wrk";                                                                                                     
    libname  e  "e:/wrk";                                                                                                     
    libname  f  "f:/wrk";                                                                                                     
    libname  i  "i:/wrk";                                                                                                     
                                                                                                                              
    /* keep this as a view ? */                                                                                               
    data allsrt/view=allsrt;                                                                                                  
      set                                                                                                                     
        c.srt1                                                                                                                
        d.srt125000001                                                                                                        
        e.srt250000001                                                                                                        
        f.srt375000001                                                                                                        
        i.srt500000001                                                                                                        
        c.srt625000001                                                                                                        
        e.srt750000001                                                                                                        
        f.srt875000000 end=dne;                                                                                               
      by distinct_pid;                                                                                                        
      if first.distinct_pid then output;                                                                                      
    run;quit;                                                                                                                 
                                                                                                                              
    /*                                                                                                                        
    NOTE: There were 15597099 observations read from the data set C.SQL_PID1.                                                 
    NOTE: There were 15595691 observations read from the data set D.SQL_PID125000001.                                         
    NOTE: There were 15596531 observations read from the data set E.SQL_PID250000001.                                         
    NOTE: There were 15599366 observations read from the data set F.SQL_PID375000001.                                         
    NOTE: There were 15595718 observations read from the data set I.SQL_PID500000001.                                         
    NOTE: There were 15596024 observations read from the data set C.SQL_PID625000001.                                         
    NOTE: There were 15593422 observations read from the data set E.SQL_PID750000001.                                         
    NOTE: There were 10613886 observations read from the data set F.SQL_PID875000000.                                         
    500 !  quit;                                                                                                              
    NOTE: PROCEDURE SQL used (Total process time):                                                                            
          real time           58.44 seconds                                                                                   
          user cpu time       56.45 seconds                                                                                   
    */                                                                                                                        
                                                                                                                              
                                                                                                                              
    /* If you want count distinct */                                                                                          
                                                                                                                              
    proc sql;                                                                                                                 
       select count(*) format=comma23. from work.allhsh                                                                       
    ;quit;                                                                                                                    
                                                                                                                              
    /*                                                                                                                        
    NOTE: The PROCEDURE SQL printed page 1.                                                                                   
    NOTE: PROCEDURE SQL used (Total process time):                                                                            
          real time           55.19 seconds                                                                                   
          cpu time            59.60 seconds                                                                                   
                                                                                                                              
                                                                                                                              
    * same as proc sql                                                                                                        
     118,339,280                                                                                                              
    */                                                                                                                        
                                                                                                                              
    * _               _                                                                                                       
     | |__   __ _ ___| |__                                                                                                    
     | '_ \ / _` / __| '_ \                                                                                                   
     | | | | (_| \__ \ | | |                                                                                                  
     |_| |_|\__,_|___/_| |_|                                                                                                  
                                                                                                                              
    ;                                                                                                                         
                                                                                                                              
    /* _r is the rooot to your 'files' folder */                                                                              
                                                                                                                              
    filename ft15f001 "&_r/oto/unq_hsh.sas";                                                                                  
    parmcards4;                                                                                                               
    %macro unq_hsh(drv,beg,end);                                                                                              
                                                                                                                              
    options source;                                                                                                           
    libname f "f:/unq";                                                                                                       
    libname drv "&drv.:/wrk";                                                                                                 
                                                                                                                              
    data  hsh&beg;                                                                                                            
       if _N_ = 1 then do;                                                                                                    
          declare hash h(hashexp:20);                                                                                         
          h.defineKey('PID');                                                                                                 
          h.defineDone();                                                                                                     
       end;                                                                                                                   
       set f.have(keep=pid firstobs=&beg obs=&end);                                                                           
       /* add code here */                                                                                                    
       if h.check() ne 0 then do;                                                                                             
          output;                                                                                                             
          h.add();                                                                                                            
       end;                                                                                                                   
    run;                                                                                                                      
                                                                                                                              
    proc sort data=hsh&beg out=drv.hash&beg noequals;                                                                         
      by pid;                                                                                                                 
    run;quit;                                                                                                                 
                                                                                                                              
    %mend unq_hsh;                                                                                                            
    ;;;;                                                                                                                      
    run;quit;                                                                                                                 
                                                                                                                              
    * test;                                                                                                                   
    %unq_hsh(c,1,61250000)                                                                                                    
                                                                                                                              
                                                                                                                              
    /* not the request for 4gb ram - better than grid? */                                                                     
                                                                                                                              
    %let _s=%sysfunc(compbl(C:\Progra~1\SASHome\SASFoundation\9.4\sas.exe -sysin c:\nul -rsasuser                             
     -sasautos c:\oto -work f:\wrk -config c:\cfg\cfgsas94m6.cfg -utilloc e:/wrk));                                           
                                                                                                                              
    systask kill sys1 sys2 sys3 sys4 sys5 sys6 sys7 sys8;                                                                     
                                                                                                                              
    systask command "&_s -altlog d:\unq\log\unq1.log -termstmt '%nrstr(%unq_hsh(c,1,125000000))'" taskname=sys1;              
    systask command "&_s -altlog d:\unq\log\unq2.log -termstmt '%nrstr(%unq_hsh(d,125000001,250000000))'" taskname=sys2;      
    systask command "&_s -altlog d:\unq\log\unq3.log -termstmt '%nrstr(%unq_hsh(e,250000001,375000000))'" taskname=sys3;      
    systask command "&_s -altlog d:\unq\log\unq4.log -termstmt '%nrstr(%unq_hsh(f,375000001,500000000))'" taskname=sys4;      
    systask command "&_s -altlog d:\unq\log\unq5.log -termstmt '%nrstr(%unq_hsh(i,500000001,625000000))'" taskname=sys5;      
    systask command "&_s -altlog d:\unq\log\unq6.log -termstmt '%nrstr(%unq_hsh(c,625000001,750000000))'" taskname=sys6;      
    systask command "&_s -altlog d:\unq\log\unq7.log -termstmt '%nrstr(%unq_hsh(e,750000001,875000000))'" taskname=sys7;      
    systask command "&_s -altlog d:\unq\log\unq8.log -termstmt '%nrstr(%unq_hsh(f,875000001,1000000000))'" taskname=sys8;     
                                                                                                                              
    waitfor sys1 sys2 sys3 sys4 sys5 sys6 sys7 sys8;                                                                          
                                                                                                                              
    /* sys6 log (8 batch programs run in parallel - longest wast sys8                                                         
                                                                                                                              
    NOTE: SAS Institute Inc., SAS Campus Drive, Cary, NC USA 27513-2414                                                       
    NOTE: The SAS System used:                                                                                                
          real time           2:33.21                                                                                         
          cpu time            2:34.86                                                                                         
    */                                                                                                                        
                                                                                                                              
    libname  c  "c:/wrk";                                                                                                     
    libname  d  "d:/wrk";                                                                                                     
    libname  e  "e:/wrk";                                                                                                     
    libname  f  "f:/wrk";                                                                                                     
    libname  i  "i:/wrk";                                                                                                     
                                                                                                                              
    /* keep this as a view ? */                                                                                               
    data allhash/view=allhash;                                                                                                
      set                                                                                                                     
        c.hash1                                                                                                               
        d.hash125000001                                                                                                       
        e.hash250000001                                                                                                       
        f.hash375000001                                                                                                       
        i.hash500000001                                                                                                       
        c.hash625000001                                                                                                       
        e.hash750000001                                                                                                       
        f.hash875000001 end=dne;                                                                                              
      by pid;                                                                                                                 
      if first.pid then output;                                                                                               
    run;quit;                                                                                                                 
                                                                                                                              
    /*                                                                                                                        
    NOTE: There were 15597099 observations read from the data set C.HASH1.                                                    
    NOTE: There were 15595691 observations read from the data set D.HASH125000001.                                            
    NOTE: There were 15596531 observations read from the data set E.HASH250000001.                                            
    NOTE: There were 15599366 observations read from the data set F.HASH375000001.                                            
    NOTE: There were 15595718 observations read from the data set I.HASH500000001.                                            
    NOTE: There were 15596024 observations read from the data set C.HASH625000001.                                            
    NOTE: There were 15593422 observations read from the data set E.HASH750000001.                                            
    NOTE: There were 10613886 observations read from the data set F.HASH875000001.                                            
    */                                                                                                                        
                                                                                                                              
                                                                                                                              
    /* If you want count distinct */                                                                                          
                                                                                                                              
    proc sql;                                                                                                                 
       select count(*) format=comma23. from work.allhash                                                                      
    ;quit;                                                                                                                    
                                                                                                                              
    /*                                                                                                                        
    NOTE: The PROCEDURE SQL printed page 1.                                                                                   
    NOTE: PROCEDURE SQL used (Total process time):                                                                            
          real time           55.19 seconds                                                                                   
          cpu time            59.60 seconds                                                                                   
                                                                                                                              
                                                                                                                              
    * same as proc sql                                                                                                        
     118,339,280                                                                                                              
    */                                                                                                                        
                                                                                                                              
