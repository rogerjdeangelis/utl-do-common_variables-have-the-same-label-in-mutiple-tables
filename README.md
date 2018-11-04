# utl-do-common_variables-have-the-same-label-in-mutiple-tables
Do common_variables have the same label in multiple tables?  

    Do common_variables have the same label in multiple tables?                                                
                                                                                                               
    Note dictionary tables are too slow on EG servers, so I suggest                                            
    you use other methods.                                                                                     
                                                                                                               
    Also I find it more useful to create a 'compare' table rather than a static report;                        
                                                                                                               
    I have benchmarked a query like the one above to take over 30 minutes                                      
    on a EG server because of the large number of preset librefs                                               
    for non-programmers.                                                                                       
                                                                                                               
    github                                                                                                     
    https://tinyurl.com/yblugj3n                                                                               
    https://github.com/rogerjdeangelis/utl-do-common_variables-have-the-same-label-in-mutiple-tables           
                                                                                                               
    SAS Forum                                                                                                  
    https://communities.sas.com/t5/SAS-Programming/merge-to-compare-labels/m-p/509693                          
                                                                                                               
                                                                                                               
                                                                                                               
    INPUT                                                                                                      
    =====                                                                                                      
                                                                                                               
       SASHELP DATASETS PRDSAL2  PRDSAL3 PRDSALE                                                               
                                                                                                               
       SASHELP.PRDSALE    Variable    Label                                                                    
                                                                                                               
                          ACTUAL      Actual Sales                                                             
                          COUNTRY     Country                                                                  
                          DIVISION    Division                                                                 
                          MONTH       Month                                                                    
                          PREDICT     Predicted Sales                                                          
                          PRODTYPE    Product type                                                             
                          PRODUCT     Product                                                                  
                          QUARTER     Quarter                                                                  
                          REGION      Region                                                                   
                          YEAR        Year                                                                     
                                                                                                               
       SASHELP.PRDSAL2    ACTUAL      Actual Sales                                                             
                          COUNTRY     Country                                                                  
                          COUNTY      County                                                                   
                          MONTH       Month                                                                    
                          MONYR       Month/Year                                                               
                          PREDICT     Predicted Sales                                                          
                          PRODTYPE    Product Type                                                             
                          PRODUCT     Product                                                                  
                          QUARTER     Quarter                                                                  
                          STATE       State/Province                                                           
                          YEAR        Year                                                                     
                                                                                                               
       SASHELP.PRDSAL3    ACTUAL      Actual Sales                                                             
                          COUNTRY     Country                                                                  
                          COUNTY      County                                                                   
                          DATE        Date                                                                     
                          MONTH       Month                                                                    
                          PREDICT     Predicted Sales                                                          
                          PRODTYPE    Product Type                                                             
                          PRODUCT     Product                                                                  
                          QUARTER     Quarter                                                                  
                          STATE       State/Province                                                           
                          YEAR        Year                                                                     
                                                                                                               
                                                                                                               
    EXAMPLE OUTPUT                                                                                             
    ==============                                                                                             
                                                                                                               
      WORK.WANT total obs=14                                                                                   
                                                                                                               
       VARIABLE    SASHELP_PRDSAL2    SASHELP_PRDSAL3    SASHELP_PRDSALE                                       
                                                                                                               
       ACTUAL      Actual Sales       Actual Sales       Actual Sales                                          
       COUNTRY     Country            Country            Country                                               
       COUNTY      County             County                                                                   
       DATE                           Date                                                                     
       DIVISION                                          Division                                              
       MONTH       Month              Month              Month                                                 
       MONYR       Month/Year                                                                                  
       PREDICT     Predicted Sales    Predicted Sales    Predicted Sales                                       
       PRODTYPE    Product Type       Product Type       Product type                                          
       PRODUCT     Product            Product            Product                                               
       QUARTER     Quarter            Quarter            Quarter                                               
       REGION                                            Region                                                
       STATE       State/Province     State/Province                                                           
       YEAR        Year               Year               Year                                                  
                                                                                                               
                                                                                                               
      WORK.LOGOUT total obs=4                                                                                  
                                                                                                               
         DSN      RC           STATUS                                                                          
                                                                                                               
       PRDSAL2     0    Meta Tasks Sucessful                                                                   
       PRDSAL3     0    Meta Tasks Sucessful                                                                   
       PRDSALE     0    Meta Tasks Sucessful                                                                   
       PRDSALE     0    Transpose Task Sucessful                                                               
                                                                                                               
                                                                                                               
    PROCESS                                                                                                    
    =======                                                                                                    
                                                                                                               
    * input data is internal;                                                                                  
                                                                                                               
    proc datasets lib=work kill;                                                                               
    run;quit;                                                                                                  
                                                                                                               
    options validvarname=upcase;                                                                               
    data logout;                                                                                               
                                                                                                               
       do dsn='PRDSAL2','PRDSAL3','PRDSALE';                                                                   
                                                                                                               
          call symputx('dsn',dsn);                                                                             
                                                                                                               
          rc=dosubl('                                                                                          
            run;quit;                                                                                          
            ods output variables=&dsn(keep=member variable label num);                                         
            proc contents data=sashelp.&dsn;                                                                   
            run;quit;                                                                                          
            proc append data=&dsn base=prdsal;                                                                 
            run;quit;                                                                                          
            %sysfunc(ifc(&dsn=PRDSALE, %str(proc sort data=prdsal;by variable member;run;quit;),));            
            %let cc1=&syserr;                                                                                  
            run;quit;                                                                                          
          ');                                                                                                  
                                                                                                               
          if symgetn('cc1') =0 then status = "Meta Tasks Sucessful     ";                                      
          else status = "Meta Tasks Failed";                                                                   
          output;                                                                                              
                                                                                                               
       end;                                                                                                    
                                                                                                               
       rc=dosubl('                                                                                             
           proc transpose data=prdsal out=want(drop=_label_);                                                  
           by variable;                                                                                        
           id member;                                                                                          
           var label;                                                                                          
           run;quit;                                                                                           
           %let cc2=&syserr;                                                                                   
           run;quit;                                                                                           
       ');                                                                                                     
                                                                                                               
       if symgetn('cc2') =0 then status = "Transpose Task Sucessful";                                          
       else status = "Transpose Task Failed";                                                                  
                                                                                                               
       output;                                                                                                 
                                                                                                               
       stop;                                                                                                   
                                                                                                               
    run;quit;                                                                                                  
                                                                                                               
                                                                                                               
                                                                                                               
                                                                                                               
                                                                                                               
