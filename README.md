LEARNING SAS


- PROGRAMAR PRIMERO EN PAPEL

Seminar: What are 10 ways to make my SAS code run more efficiently?

- Agregar comentarios para hacer más fácil la lectura del código
    - /*Comment*/
    - *Comment;
- Escribir el código indentado (Format code or Ctrl-i)
    DATA new;
      SET old;
    RUN;
    PROC MEANS DATA=new;
      VAR newyear;
      CLASS year;
    RUN;
- Usar nombres que sea descriptivos
    DATA salaryinfo2022;
      SET salaryinfo2021;
      newsalary=oldsalary+increase;
    RUN;
- Reduce observations and variables (unnecessary data)
- Reduce the number of times the data is processed
- Poner GLOBALS al principio
- Minimizar el número de veces que se leen los datos en un procedimiento cuando sea posible para no llenar la memoria y hacer los procesos más rápidos y eficientes, tener en cuenta que cada Data Step es un loop over each observation.
- Limitar el número de veces que se ordenan los datos, se puede incluir la opción PRESORTED en PROCSORT cuando se sospecha que los datos ya están ordenados, está comprobación corre más rapido que ordenar los datos. The option TAGSORT with PROCSORT for large datasets can reduce the space used but it might take longer (because it goes twice for the data). Use BY statements with sorted data without using PROC SORT.
- Usar IF, THEN, ELSE, ELSE IF, en lugar de IF IF IF
- Ordenar las condiciones IF THEN en orden de probabilidad
- Seleccionar solo las columnas que necesito (it might seem obvious but it is not), usar KEEP y DROP en la línea de SET 
- Si estoy usando SAS data WHERE es más eficiente que IF
    - Added efficiency: IF loads the whole database, WHERE subsets first. When using SAS/Access engines, SAS attempts to send the WHERE clause to the RDBMS (relational database management system) for evaluation rather than to SAS; With IF statement, SAS must do the processing.
- Cuando se hace MERGE:
    - Si uso WHERE el sub-setting ocurre antes del merge
    - Si uso IF el sub-setting ocurre después del merge
- Cuando se hace el append:
    - Si uso SET estoy procesando todas las observaciones de la base original y la base que quiero agregar, cuando uso PROC solo estoy procesando las observaciones de la base que quiero agregar
- Variables that are less than 3 bytes I can make them character to save space if I am not going to do math with them. Example dummies.
- RESUSE=YES and COMPRESS=YES or COMPRESS=BINARY(might make the job run longer)
- Delete WORK SAS data sets as soon as they are no longer needed


Truncation in Binary Numbers
The decimal value `1/10` has a finite decimal representation (0.1), but in binary it has an infinitely repeating representation. In binary, the value converts to
`0.000110011001100110011 ...`

![](https://paper-attachments.dropbox.com/s_838766B75B7717404C528139F5BBCED8D6BCF674885F88AE209B072CC261F8BF_1654353885150_image.png)



    data a;
      point_three=0.3;
      three_times_point_one= 3 * 0.1;
      difference= point_three - three_times_point_one;
    put 'The difference is ' difference;
    run;

Computational Considerations
Regardless of how much precision is available, there are still some numbers that cannot be represented exactly. Most rational numbers (for example, `.1`) cannot be represented exactly in base 2 or base 16. This is why it is often difficult to store fractions in floating-point representation.

![](https://paper-attachments.dropbox.com/s_838766B75B7717404C528139F5BBCED8D6BCF674885F88AE209B072CC261F8BF_1654442776395_Untitled.png)


For example, consider the following DATA step:

    data _null_;
       do i=-1 to 1 by .1;
          put i=;
          if i=0 then put 'AT ZERO';
       end;
    run;

The `AT ZERO` message in the DATA step is never printed because the accumulation of the imprecise number introduces enough errors that the exact value of 0 is never encountered. The calculated result is close to 0, but never exactly equal to 0. Therefore, when numbers cannot be represented exactly in floating point, performing mathematical operations with other non-exact values can compound the imprecision.


![](https://paper-attachments.dropbox.com/s_838766B75B7717404C528139F5BBCED8D6BCF674885F88AE209B072CC261F8BF_1654443179820_Untitled.png)


Using the ROUND Function to Avoid Computational Errors

    data _null_;
       do i=-1 to 1 by .1;
          i = round(i,.1);
          put i=;
          if i=0 then put 'AT ZERO';
       end;
    run;

Using Formats to Confirm Precision Errors

    data a;    
    x=15.7-11.9;   
    if x=3.8 then put 'equal';      
       else put 'not equal';    
    put x=10.8;   
    put x=18.16; 
    put x=hex16.;
    run;

not equal
x=3.80000000
x=3.7999999999999900
x=400E666666666664

Using the ROUND Function to Avoid Comparison Errors

    data _null_;
       x=1/3;
       if round(x,.00001)=.33333 then put 'MATCH';
    run;

Using the LENGTH Statement When Comparing Values

    data temp;    
      length x 3;    
      x=257;     
    run;  
    
    data _null_;    
    set temp;    
    if x=257 then put 'FOUND';    
    run;

Determining How Many Bytes Are Needed to Store a Number Accurately
You can use the TRUNC function to determine the minimum number of bytes that are needed to store a value accurately. The following program finds the minimum length of bytes (MinLen) that are needed for numbers stored in a native SAS data set named Numbers in an IBM mainframe environment. The data set Numbers contains the variable Value. Value contains a range of numbers from `269` to `272:`

    data numbers;    
    input value;    
    datalines; 
    269 
    270 
    271 
    272 
    ; 
    
    data temp;    
       set numbers;   
        x=value;    
       do L=8 to 1 by -1;       
       if x NE trunc(x,L) then       
          do;          
             minlen=L+1;          
             output;          
             return;       
          end;   
       end; 
    run;  
    
    proc print noobs;   
       var value minlen;
     run;
DATA STEP

A DATA step is processed in two-phase sequences:
Compilation phase: Each statement is scanned for syntax errors
Execution phase: the DATA step reads and processes the input data


INFILE

Identifies the location of the external file, specifies an external file to read with an INPUT statement.

        DSD (Delimiter Sensitive Data): permite que si por ejemplo tengo datos que estan asi:
        1,2,”a,m” lea esto como 3 variables y no 4, quita las comillas, y dice que el delimitador es coma. 
        DLM: permite especificar un delimitador diferente a coma. The delimiter is case sensitive. Some common delimiters are the comma (,), verticle pipe (|), semicolon(;), and tab. The tab is specified by a hexadecimal character, '09'x.
        FLOWOVER: is the default behavior. The DATA step simply reads the next record into the input buffer, attempting to find values to assign to the rest of the variable names in the INPUT statement.
        STOPOVER: causes the DATA step to stop processing if an INPUT statement reaches the end of the current record without finding values for all variables in the statement. Use this option if you expect all of the data in the external file to conform to a given standard and if you want the DATA step to stop when it encounters a data record that does not conform to the standard.
        MISSOVER: prevents the DATA step from going to the next line if it does not find values in the current record for all of the variables in the INPUT statement. Instead, the DATA step assigns a missing value for all variables that do not have values.
        TRUNCOVER: causes the DATA step to assign the raw data value to the variable even if the value is shorter than expected by the INPUT statement. If, when the DATA step encounters the end of an input record, there are variables without values, the variables are assigned missing values for that observation.
        END=variable: specifies a variable that SAS sets to 1 when the current input data record is the last in the input file. Until SAS processes the last data record, the END= variable is set to 0. Like automatic variables, this variable is not written to the data set.
        
        
    INPUT: instruct SAS how to read observations. Describes the arrangement of values in the input data record and assigns input values to the corresponding SAS variables.
    LENGHT: makes sure that the data is not truncated. Specifies the number of bytes for storing character and numeric variables, or the number of characters for storing VARCHAR variables.
    If you have not explicitly specified the number of storage bytes, then SAS uses the default length of 8 bytes, and the maximum integer then depends solely on what operating system you are using. Use the LENGTH statement to truncate values only when disk space is limited
![](https://paper-attachments.dropbox.com/s_838766B75B7717404C528139F5BBCED8D6BCF674885F88AE209B072CC261F8BF_1654353974839_image.png)

    Always store real numbers in the full 8 bytes of storage. If you want to save disk space by using the LENGTH statement to reduce the length of your variables, you can do so but only for variables whose values are integers. When adjusting the length of variables, make sure that the values are less than or equal to the largest integer allowed for that specified length.
    
    DATA myData;
      LENGTH num 3;
         num=8000;
    RUN;
    
    RETAIN: causes a variable that is created by an INPUT or assignment statement to retain its value from one iteration of the DATA step to the next.
    FORMAT : 
    - Formats are used to change the way values are displayed in data and reports.
    - Formats do not change the underlying data values.
    - Formats can be applied in a procedure using the FORMAT statement.
     
    INFORMAT 🙃: Todavía tengo dudas, sirve para la variable fecha que en algunos casos es mm/dd/yy y en otros es dd/mm/yy


    DATA ex3_1;
    INFILE '..\example3_1.txt';
    INPUT name $ 1-7 height 9-10 weight 12-14;
    BMI = 700weight/(heightheight);
    OUTPUT;
    RUN;


|              | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 | 11 | 12 | 13 | 14 | 15 |
| ------------ | - | - | - | - | - | - | - | - | - | -- | -- | -- | -- | -- | -- |
| Input buffer | B | A | R | B | A | R | A |   | 5 | 3  |    | 1  | 2  | 5  |    |

Program Data Vector (PDV)

It is a logical area in memory where SAS builds a data set, one observation at a time. When a program executes, SAS reads data values from the input buffer or creates them by executing SAS language statements. The data values are assigned to the appropriate variables in the program data vector. From here, SAS writes the values to a SAS data set as a single observation.
Along with data set variables and computed variables, the PDV contains two automatic variables, _N_ and _ERROR_. 

- The _N_ variable counts the number of times the DATA step begins to iterate. 
- The _ERROR_ variable signals the occurrence of an error caused by the data during execution. The value of _ERROR_ is either 0 (indicating no errors exist), or 1 (indicating that one or more errors have occurred). SAS does not write these variables to the output data set.


| N (D) | ERROR(D) | NAME(K) | WEIIGHT(K) | HEIGHT(K) | IBM(K) |
| ----- | -------- | ------- | ---------- | --------- | ------ |
|       |          |         |            |           |        |

(D): Dropped
(K): Kept

It runs like a loop by observation, N increases and ERROR is 1 is there is a missing in any of the variables.
When reading data from a SAS format it will be read directly in the PDV, it won't be read first in the Input buffer.


    DATA ex3_1;
    set example1;
    BMI = 700weight/(heightheight);
    OUTPUT;
    RUN;

SAS sets each variable to missing in the PDV at the beginning of every iteration of the execution

Con la opción _NULL_ puedo hacer un Data Step sin necesidad de crear una base en SAS.

    %LET dsn=SASHELP.CARS;
    DATA _NULL_;
      SET sashelp.vcolumn end=last;
      WHERE CATX('-', libname, memname)="&dsn";
      IF type='char' THEN charcount+1;
      ELSE IF type='num' THEN numcount+1;
      IF last=1 then do;
        CALL SYMPUTX('ccount', charcount);
        CALL SYMPUTX('ncount', numcount);
      END;
    RUN;
WRITING LOOPS IN THE DATA STEP

It is more complex in SAS because of the existence of implicit and explicit loop~
IMPLICIT LOOPS
The DATA step works like a loop -an implicit loop
During the execution of the DATA step processing, the DATA step works like a loop, repetitively reading the data and creating observations one at a time. This type of loop is referred to as an implicit loop.
It repetitively executes statements, reads data values and creates observations in the PDV one at a time.
Each loop is called an iteration

The RANUNI function: 
RANUNI(seed)

    It generates a number distributed Uniform (0,1) e.g. 0.125151 0.35554 0.568412, etc
    SEED is a nonnegative integer
    The RANUNI function generates a stream of numbers based on SEED
    When SEED is set to 0, the generated number cannot be reproduced
    When SEED is a non-zero number, the generated number can be produced~

**

    DATA ex5_1 (drop=rannum);
      SET patient;
      rannum = rannuni(2);
      IF rannum > 0.5 THEN GROUP = 'D';
      ELSE group='P';
    RUN;

EXPLICIT LOOPS
This is an example of a long way to run an explicit loop:

    DATA ex5_2(drop=rannum);
    ID='M2390';
    rannum=RANUNI(2);
    IF rannum>0.5 THEN group='D';
    ELSE group='P';
    OUTPUT;
    
    ID='F2390';
    rannum=RANUNI(2);
    IF rannum>0.5 THEN group='D';
    ELSE group='P';
    OUTPUT;
    
    ID='F2340';
    rannum=RANUNI(2);
    IF rannum>0.5 THEN group='D';
    ELSE group='P';
    OUTPUT;
    
    ID='M1240';
    rannum=RANUNI(2);
    IF rannum>0.5 THEN group='D';
    ELSE group='P';
    OUTPUT;
    RUN;

We can rewrite the previous code with the DO command
General form for an iterative DO loop:

    DO index-variable=value1, value2, ... valuen;
        SAS statements
        END;
        INDEX-VARIABLE: contains the value of the current iteration

The loop will execute along VALUE1 through VALUEN
The VALUES can be either character or numeric~


    DATA ex5_2(drop=rannum);
    DO ID='M2390','F2390','F2340','M1240';
    rannum=RANUNI(2);
    IF rannum>0.5 THEN group='D';
    ELSE group='P';
    OUTPUT;
    END;
    RUN;

EXPLICIT ITERATIVE LOOP
Usually we use iterative DO loop and loop along a sequence of integers

    DO index-variable=startTOstop;
    SAS statements
    END;


    DATA ex5_2(drop=rannum);
    DO ID=1 to 4;
    rannum=RANUNI(2);
    IF rannum>0.5 THEN group='D';
    ELSE group='P';
    OUTPUT;
    END;
    RUN;

EXPLICIT DO WHILE LOOP


    DO WHILE (expression);
    SAS statements
    END;

The loop will not continue if the EXPRESSION is false


    DATA ex5_2(drop=rannum);
    DO WHILE (id<4); /It starts with 0 by default/
    id+1 /It adds 1 so the fisrt ID in the OUTPUT is 1/
    rannum=RANUNI(2);
    IF rannum>0.5 THEN group='D';
    ELSE group='P';
    OUTPUT;
    END;
    RUN;

EXPLICIT DO UNTIL LOOP 

    DO UNTIL (expression);
    SAS statements
    END;

The loop will not continue for another iteration if the EXPRESSION
is true


    DATA ex5_2(drop=rannum);
    DO UNTIL (id>=4); /It starts with 0 by default/
    id+1 /It adds 1 so the fisrt ID in the OUTPUT is 1/
    rannum=RANUNI(2);
    IF rannum>0.5 THEN group='D';
    ELSE group='P';
    OUTPUT;
    END;
    RUN;

NESTED LOOP 
A loop inside of another loop


    DATA ex5_7;
    LENGTH center $4;
    DO center="COH", "UCLA", "USC";
    DO id=1 to 4;
    IF RANUNI(2)>0.2 THEN group='D';
    ELSE group='P';
    OUTPUT;
    END;
    END;
    RUN;

COMBINING IMPLICIT AND EXPLICIT LOOPS


    DATA trial7;
    set cancer_center;
    DO ID=1 to 4;
    IF RANUNI(2)>0.5 THEN group='D';
    ELSE group='P';
    OUTPUT;
    END;
    RUN;


BASICS DATA MANAGEMENT

The extension sas7bdat is a SAS table.
Accessing Data through Libraries

- A libref is the name of the library that can be used in a SAS program to read data files.
- The engine provides instructions for reading SAS files and other types of files.
- The path provides the directory where the collection of tables is located.
- The libref remains active until you clear it, delete it, or shut down SAS.
    LIBNAME libref engine "path";
    LIBNAME libref CLEAR;

Automatic Libraries

- Tables stored in the Work library are deleted at the end of each SAS session.
- Work is the default library, so if a table name is provided in the program without a libref, the table will be read from or written to the Work library.
- The Sashelp library contains a collection of sample tables and other files that include information about your SAS session.


IMPORT DATA 

Import DATA SAS file

    %PUT My version of SAS is %sysvlong;
    LIBNAME DatosSas "C:\Users\kthe\Carpeta";
    DATA GEIH.tempdata;
        SET GEIH.misdatos;
    RUN;

Import DATA EXCEL XLSX


    LIBNAME DatosExcel xlsx "C:\Users\kthe\Carpeta\misdatos.xlsx";
    /*It creates a database for each sheet in the excel*/;
    OPTION VALIDVARNAME=v7;

VALIDVARNAME: SAS System Option (for reading EXCEL)
Controls the type of SAS variable names that can be used or created during a SAS session

    V6:Indicates that a DBMS column name is changed to a valid SAS name, it has limitations.
    V7: Indicates that a DBMS column name is changed to a valid SAS name.
        - This is the default:
        - Up to 32 mixed-case alphanumeric characters are allowed.
        - Names must begin with an alphabetic character or an underscore.
        - Invalid characters are changed to underscores.
        - Any column name that is not unique when it is normalized is made unique by appending a counter (0, 1, 2, and so on) to the name.
    UPPERCASE: Indicates that a DBMS column name is changed to a valid SAS name as described in V7 except that variable names are in uppercase.
    ANY: allows any characters in DBMS column names to appear as valid characters in SAS variable names. Symbols, such as the equal sign (=) and the asterisk (*), must be contained in a 'variable-name'n construct. Column names that do not follow the SAS naming conventions. (DO NOT RECOMMEND)


    PROC CONTENTS DATA=DatosExcel._ALL_ NODS;
    RUN;
    /*NODS suppresses printing the contents of individual files when you specify _ALL_ in the DATA= option. The CONTENTS statement prints only the SAS data library directory.*/
    
    PROC PRINT DATA=DatosExcel.Sheet1;
    RUN;
    
    PROC PRINT DATA=DatosExcel.Sheet1;
        WHERE var1="1"
        TITLE "Variable 1 es igual a 1"
    RUN;
    
    DATA Sheet1_A Sheet1_B;
        SET DatosExcel.Sheet1;
        IF var2="A" THEN OUTPUT Sheet1_A;
        ELSE IF var2="B" THEN OUTPUT Sheet1_B;
    RUN;
    
    PROC PRINT DATA=Sheet1_A;
        TITLE "Datos en la hoja 1 para A";
    RUN;
    
    PROC PRINT DATA=Sheet1_B;
        TITLE "Datos en la hoja 1 para B";
    RUN;
    
    PROC IMPORT DATAFILE="C:\Users\kthe\Carpeta\misdatos.xlsx"
        DBMS=XLSX
        OUT=WORK.Sheet1 REPLACE;
        SHEET=Sheet1;
    RUN;
    
    PROC SQL;
        SELECT * FROM Sheet1;
    QUIT;

PROC - Import DATA from CSV format


    %LET path=C:\Users\kthe\Carpeta;
    OPTIONS VALIDVARNAME=v7
    PROC IMPORT DATAFILE="&path\cars.csv"
                OUT=cars
                DBMS=csv REPLACE;
    RUN;

PROC IMPORT scans the first 20 rows and by scanning the first 20 rows of each column it is going to make an assumption on what type of column it is and length it needs.

What PROC IMPORT  did was: it used a DATA STEP with an INFILE statement.


    PROC IMPORT DATAFILE="&path\cars.csv"
                OUT=cars
                DBMS=csv REPLACE;
                GUESSINGROWS=max; /*This helps with the truncation but it is very slow to run when having a big database*/
                GETNAMES=YES
    RUN;


- The DBMS option identifies the file type. The CSV value is included with Base SAS.
- The OUT= option provides the library and name of the SAS output table.
- The REPLACE option is necessary to overwrite the SAS output table if it exists.
- SAS assumes that column names are in the first line of the text file and data begins on the second line.
- Date values are automatically converted to numeric SAS date values and formatted for easy interpretation.
- The GUESSINGROWS= option can be used to increase the number of rows SAS scans to determine each column’s type and length from the default 20 rows up to 32,767.
EXPORT DATA 

DATA STEP - Export DATA from SAS to XLSX

Aquí lo que hago es: tengo un archivo de Excel "misdatos.xlsx".
Lo subo a SAS en su propia librería.
Con DATA STEP llamo la base CLASS que ya está en SAS, le creo una variable AgeCat, y agrego la base CLASS como una hoja excel adicional a "misdatos.xlsx"

    LIBNAME DatosExcel "C:\Users\kthe\Carpeta\misdatos.xlsx" 
    %PUT My version of SAS is %sysvlong;
    OPTION VALIDVARNAME=v7;
    
    DATA DatosExcel.class;
        SET sashelp.class;
        IF age LT 13 THEN AgeCat=1;
        ELSE IF 13 <= age <=14 AgeCat=2;
        ELSE IF age GE 15 AgeCat=3;
    RUN;
    /*LT:  LESS THAN */
    /*GE:  GREATER THAN OR EQUAL TO */
    
    PROC PRINT DATA=x.class;
    RUN;
    
    LIBNAME X CLEAR;

Export DATA from SAS to XLSX PROC
Aquí lo que hago es: abro de nuevo el archivo Excel "misdatos.xlsx".
Lo subo a SAS en su propia librería.
Llamo la base SNACKS que ya está en SAS.
Con PROC se crea una base pequeña con Snacks.
CON PROC agrego la base Snacks como una hoja excel adicional a "misdatos.xlsx"


    LIBNAME DatosExcel "C:\Users\kthe\Carpeta\misdatos.xlsx" 
    
    PROC SORT DATA=sashelp.snacks(keep=Product Price)
        OUT x.snacks NODUPKEY;
    BY product;
    RUN;
    
    PROC PRINT DATA=x.snacks NOOBS N;
    title 'Yummy Snacks';
    RUN;
    
    LIBNAME x CLEAR;

Otra opción con PROC cuando quiero crear un archivo nuevo en EXCEL


    /*Con PROC EXPORT pongo los datos CARS directamente en Excel*/
    
    PROC EXPORT DATA=sashelp.cars(drop=Invoice);
        OUTFILE="C:\Users\kthe\Carpeta\cars.xlsx"
        DBMS=XLSX REPLACE;
    RUN;
    
    /*SAS allows 32 characters long variables*/
    /*EXCEL allows 31 characters long variables*/


DATA STEP - SORTING DATA AND REMOVING DUPLICATES


- PROC SORT sorts the rows in a table on one or more character or numeric columns.
- The OUT= option specifies an output table. Without this option, PROC SORT changes the order of rows in the input table.
- The BY statement specifies one or more columns in the input table whose values are used to sort the rows. By default, SAS sorts in ascending order.
- The DUPOUT= option creates an output table containing duplicates removed.
- The NODUPKEY option keeps only the first row for each unique value of the column(s) listed in the BY statement.  
- Using _ALL_ in the BY statement sorts by all columns and ensures that entirely duplicate rows are adjacent in the sorted table and are removed. 


    PROC SORT DATA=input-table <OUT=output-table>;
            BY <DESCENDING> col-name(s);
    RUN;


    PROC SORT DATA=input-table <OUT=output-table>
                          NODUPKEY <DUPOUT=output-table>;
            BY <DESCENDING> col-name(s);
    RUN;


    PROC SORT DATA=input-table <OUT=output-table>  
                          NODUPKEY <DUPOUT=output-table>;
            BY _ALL_;
    RUN;


DATA STEP - BY GROUP

BY-group processing method:


    PROC SORT DATA=b;
    by by_variable;
    RUN;
    DATA a;
    set b;
    by by_variable;
    ...
    ...
    RUN;

For each BY-variable, SAS created two temporary variables:

    FIRST.VARIABLE
    LAST.VARIABLE

FIRST.VARIABLE & LAST.VARIABLE are set to 1 at the beginning of the execution phase
They are not being output to the final dataset

| ID  | TIME | SCORE | GROUPING | FIRST.ID | LAST.ID |
| --- | ---- | ----- | -------- | -------- | ------- |
| A01 | 1    | 3     | 1        | 1        | 0       |
| A01 | 2    | 4     | 1        | 0        | 0       |
| A01 | 3    | 3     | 1        | 0        | 1       |
| A02 | 1    | 4     | 2        | 1        | 0       |
| A02 | 3    | 2     | 2        | 0        | 1       |

    PROC SORT DATA=sas4_1;
    BY id;
    RUN;
    DATA sas4_2 (drop = score);
    set sas4_1;
    BY id;
    IF first.id then total=0;
    total+score;
    IF last.id; /*Retains the last observation*/;
    RUN;

This is pretty much like collapse by.
Calculating the mean instead:


    DATA sas4_2 (drop = score);
    set sas4_1;
    BY id;
    IF fisrt.id THEN DO;
    total=0;
    n=0;
    END;
    total+score;
    n+1;
    IF last.id then do;
    mean_score=total/n;
    OUTPUT;
    END;
    RUN;

OBTAINING THE MOST RECENT NON-MISSING DATA WITHIN EACH BY-GROUP


    DATA patients_single (drop=visit tgl smoke);
    SET patients_sort;
    BY patid;
    RETAIN tgl_new smoke_new;
    IF first.patid THEN DO;
    tgl_new=.;
    smoke_new="";
    END;
    IF NOT MISSING(tgl) THEN tgl_new=tgl;
    IF NOT MISSING(smoke) THEN smoke_new=smoke;
    IF LAST.patid;
    RUN;


FILTERING ROWS
- The WHERE statement is used to filter rows. If the expression is true, rows are read. If the expression is false, they are not.
- Character values are case sensitive and must be in quotation marks.
- Numeric values are not in quotation marks and must only include digits, decimal points, and negative signs.
- Compound conditions can be created with AND or OR.
- The logic of an operator can be reversed with the NOT keyword.
- When an expression includes a fixed date value, use the SAS date constant syntax: “ddmmmyyyy”d, where dd represents a 1- or 2-digit day, mmm represents a 3-letter month in any case, and yyyy represents a 2- or 4-digit year.


    PROC procedure-name ... ;
            WHERE expression;
    RUN;

WHERE Operators

> = or EQ
> ^= or ~= or NE
> > or GT
> < or LT
> >= or GE
> <= or LT

IN Operator

> WHERE col-name IN(value-1<...,value-n>);
> WHERE col-name NOT IN (value-1<…,value-n>);
> 

Special WHERE Operators

> WHERE col-name IS MISSING;
> WHERE col-name IS NOT MISSING;
> WHERE col-name IS NULL;
> WHERE col-name BETWEEN value-1 AND value-2;
> WHERE col-name LIKE "value";
> WHERE col-name =* "value";
> 

Filtering Rows with Macro Variables

> %LET macro-variable=value;
> 
> Example WHERE Statements with Macro Variables:
> WHERE numvar=&macrovar;
> WHERE charvar="&macrovar";
> WHERE datevar="&macrovar"d



DATA STEP - READING AND FILTERING DATA

Creating a copy of data

    DATA output-table;
            SET input-table;
    RUN;

Filtering rows in the DATA step

    DATA output-table;
            SET input-table;
            WHERE expression;
    RUN;

Specifying columns to include in the output data set

    DROP col-name <col-name(s)>;
    KEEP col-name <col-name(s)>;

Formatting columns in the DATA step

    DATA output-table;
            SET input-table;
            FORMAT col-name format;
    RUN;

Computing New Columns

- Using expressions to create new columns
    DATA output-table;
            SET input-table;
            new-column = expression;
    RUN;


    - The name of the column to be created or updated is listed on the left side of the equals sign.
    - Provide an expression on the right side of the equal sign.
    - SAS automatically defines the required attributes if the column is new – name, type, and length.
    - A new numeric column has a length of 8.
    - The length of a new character column is determined based on the length of the assigned string.
    - Character strings must be quoted and are case sensitive.


- Creating character columns:
    LENGTH char-column $ length;


DATA STEP - USING FUNCTIONS IN EXPRESSIONS
    function(argument1, argument 2, ...);


    DATA output-table;
            SET input-table;
            new-column=function(arguments);
    RUN;

Functions for calculating summary statistics (ignore missing values):

| SUM(num1, num2, ...)    | calculates the sum        |
| ----------------------- | ------------------------- |
| MEAN(num1, num2, ...)   | calculates the mean       |
| MEDIAN(num1, num2, ...) | calculates the median     |
| RANGE(num1, num2, ...)  | calculates the range      |
| MIN(num1, num2, ...)    | calculates the minimum    |
| MAX(num1, num2, ...)    | calculates the maximum    |
| N(num1, num2, ...)      | calculates the nonmissing |
| NMISS(num1, num2, ...)  | calculates the missing    |

Character functions:

| UPCASE(char1)<br>LOWCASE(char1)  | changes letters in a character string to uppercase or lowercase                           |
| -------------------------------- | ----------------------------------------------------------------------------------------- |
| PROPCASE(char1)                  | changes the first letter of each word to uppercase and other letters to lowercase         |
| CATS(char1, char2, ...)          | concatenates character strings and removes leading and trailing blanks from each argument |
| SUBSTR(char, position, <length>) | returns a substring from a character string                                               |

Date functions that extract information from SAS date values:

| MONTH(sas-date-value)   | returns a number from 1 through 12 that represents the month                     |
| ----------------------- | -------------------------------------------------------------------------------- |
| YEAR(sas-date-value)    | returns the four-digit year                                                      |
| DAY(sas-date-value)     | returns a number from 1 through 31 that represents the day of the month          |
| WEEKDAY(sas-date-value) | returns a number from 1 through 7 that represents the day of the week (Sunday=1) |
| QTR(sas-date-value)     | returns a number from 1 through 4 that represents the quarter                    |

Date functions that create SAS date values:

| TODAY()                          | returns the current date as a numeric SAS date value                                                                                     |
| -------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------- |
| MDY(month, day, year)            | returns SAS date value from month, day, and year values                                                                                  |
| YRDIF(startdate, enddate, ‘AGE’) | calculates a precise age between two dates. There are various values for the third argument. However, "AGE" should be used for accuracy. |

MISSING AND NOT MISSING


CONDITIONAL PROCESSING

Conditional processing with IF-THEN logic:


    IF expression THEN statement;

Conditional processing with IF-THEN-ELSE:


    IF expression THEN statement;
    <ELSE IF expression THEN statement;>
    <ELSE IF expression THEN statement;>
    ELSE statement;

Processing multiple statements with IF-THEN-DO:


    IF expression THEN DO;
       <executable statements>
    END;
    <ELSE IF expression THEN DO;
       <executable statements>
    END;
    ELSE DO;
       <executable statements>
    END;
- After the IF-THEN-DO statement, list any number of executable statements.
- Close each DO block with an END statement.

DATA STEP -IF STATEMENT EXAMPLE:
If the EXPRESSION is false:

- no further statements are processed for that observation
- the current observation is not writter to the data set
- the remaining program statements in the DATA step are not executed
- SAS immediately returns to the beginning of the DATA step


    DATA ex3_4;
    SET sas3_1;
    total+score;
    IF not missing(score);
    RUN;
    TITLE "Keep observations only when SCORE is not missing";
    PROC PRINT DATA ex3_4;
    RUN;


    DATA ex3_4;
    SET sas3_1;
    total+score;
    IF not missing(score) THEN DELETE;
    RUN;


    DATA total_score(keep = total n)
    SET sas3_1 END=LAST;
    total + score;
    n + 1;
    IF LAST;
    RUN;
    TITLE "Only keep the last observation"
    PROC PRINT DATA=total_score;
    RUN;

CASE 
PROC FORTMAT with CNTLIN 
CLASS with PROC SUMMARY or PROC MEANS with unsorted data set



Retaining the value of a newly created variable


    DATA ex3_2;
    set sas2_1;
    retatin total 0;
    total = sum(total, score);
    RUN;
| ID | SCORE | TOTAL |
| -- | ----- | ----- |
| A1 | 3     | 3     |
| A2 | .     | 3     |
| A2 | 4     | 7     |

    DATA ex3_2;
    set sas2_1;
    total + score;
    RUN;

DATA STEP - FROM WIDE FORMAT TO LONG FORMAT

Wide

| ID  | S1 | S2 | S3 |
| --- | -- | -- | -- |
| A01 | 3  | 4  | 5  |
| A02 | 4  | .  | 2  |



    DATA LONG (drop s1-s3);
    SET wide;
    time=1;
    score=s1;
    IF NOT MISSING(score) THEN OUTPUT;
    time=2;
    score=s2;
    IF NOT MISSINGS(score) THEN OUTPUT;
    time=3;
    score=s3;
    IF NOT MISSING(score) THEN OUTPUT;
    RUN;

Long

| ID  | TIME | SCORE |
| --- | ---- | ----- |
| A01 | 1    | 3     |
| A01 | 2    | 2     |
| A01 | 3    | 3     |
| A02 | 1    | 4     |
| A02 | 3    | 2     |

DATA STEP - FROM LONG FORMAT TO WIDE FORMAT


    PROC SORT DATA=long;
    BY id time;
    RUN;
    DATA wide (drop=time score);
    SET long;
    BY id;
    RETAIN s1-s3;
    IF FIRST.id THEN DO;
    s1=. ; s2=. ; s3=. ;
    END;
    IF time=1 THEN s1=score;
    ELSE IF time=2 THEN s2=score;
    ELSE s3=score;
    IF last.id;
    RUN;


DATA STEP - ARRAY PROCESSING

It is a temporary grouping of SAS variables that are arranged in a particular order and identified by an array-name. 
The array exists only for the duration of the current DATA step. 
The array-name distinguishes it from any other arrays in the same DATA step; it is not a variable.

Defining and Referencing One-dimensional Arrays
Utilizing array processing allows you to reduce the amount of coding in the DATA step. 


    ARRAY array-name[subscript]<$>
        <(initial-value-list)>;
- It does not become part of the output data
- It must be a SAS name
- It cannot be the name of a SAS variable
- It cannot be used in the LABEL, FORMAT, DROP, KEEP or LENGTH statements
- The [subscript] component describes the number of array elements.
- The simple form is specifying the dimensional size of the array.
- $ indicates that the elements in the array are character elements
- $ is not necessary if the elements have been previously defined as character elements
- If the lenghts of array elements have not been previously specified, you can use the lenght option

ARRAY-ELEMENTS 
Array elements are the variables to be included in the array
Must either be all numeric or characters
You can also provide a range of numbers are [subscript]

    ARRAY sbparry[1990:19930] sbp1990 sbp1991 sbp1992 sbp1993;

You can use an aterisk (*) as SUBSCRIPT

    ARRAY sbparry[*] sbp1 sbp2 sbp3 sbp4 sbp5 sbp6;

You must include ELEMENTS
SUBSCRIPT can be enclosed in parentheses, braces, or brackets

    ARRAY sbparry(6) sbp1 sbp2 sbp3 sbp4 sbp5 sbp6;


    ARRAY sbparry{6} sbp1 sbp2 sbp3 sbp4 sbp5 sbp6;


    ARRAY sbparry[6] sbp1 sbp2 sbp3 sbp4 sbp5 sbp6;

If ARRAY-ELEMENTS are not specified, the array elements will be implied to the variables based on names that contain array names


    ARRAY sbp[6];
    ARRAY sbp[6] sbp1 sbp2 sbp3 sbp4 sbp5 sbp6;

~NUMERIC: all numeric variables
CHARACTER: all character variables
ALL: all the variables; variables must be either all numeric or character~


    ARRAY num[] numeric;
    ARRAY char[] character;
    ARRAY allvar[*] all;

You can assign initial values to the array elements~


    ARRAY num[3] n1 n2 n3 (1 2 3);
    ARRAY chr[3] $ ('A','B','V');

TEMPORARY ARRAYS

- Contain temporary data elements, 
- They are not output to the output dataset, 
- Using temporary arrays is useful when you want to create an array only for calculation purposes, 
- When referring to a temporary data element, 
- You refer to it by the ARRAYNAME and its SUBSCRIPT, 
- You cannot use the asterisk(*) with temporary arrays,
- They are always automatically retained,
- To create a temporary array, you need to use the keyword TEMPORARY ~


    ARRAY num[3] temporary (1 2 3);

To reference an array element:

    array-name[subscript]

It must be within the lower and upper bounds of the DIMENSION of the array~

LONG WAY WITHOUT ARRAY


    DATA ex6_1;
    SET sbp;
    IF sbp1=999 THEN sbp1=.;
    IF sbp2=999 THEN sbp2=.;
    IF sbp3=999 THEN sbp3=.;
    IF sbp4=999 THEN sbp4=.;
    IF sbp5=999 THEN sbp5=.;
    IF sbp6=999 THEN sbp6=.;
    RUN;

SHORT WAY WITH ARRAY


    DATA ex6_1 (drop=i);
    SET sbp;
    ARRAY sbparray[6] sbp1 sbp2 sbp3 
    sbp4 sbp5 sbp6;
    DO i=1 to 6;
    IF sbparray[i]= 999 THEN sbparray[i]=.;
    END;
    RUN;

Optional notation 1


    DATA ex6_1 (drop=i);
    SET sbp;
    ARRAY sbparray[6] sbp1-sbp6;
    DO i=1 to 6;
    IF sbparray[i]= 999 THEN sbparray[i]=.;
    END;
    RUN;

Optional notation 2
For this one sbp1, sbp2, sbp3, sbp4, sbp5, sbp6 must already exist in the dataset


    DATA ex6_1 (drop=i);
    SET sbp;
    ARRAY sbp[6];
    DO i=1 to 6;
    IF sbp[i]= 999 THEN sbp[i]=.;
    END;
    RUN;


The DIM, HBOUND, and LBOUND Functions

The DIM function 
It returns the number of elements in a specified dimension.
It is convenient when you use NUMERIC, CHARACTER, ALL as array elements.

    DIM (array-name)
    DIM(array-name, bound-n)

If the N value is not specified, the DIM function will return the number of elements in the first dimension of the array.
The BOUND-N is either a numeric constant, variable, or SAS expresion

The HBOUND function
It returns the upper bound in a specific dimension:

    HBOUND<n>(array-name)
    HBOUND(array-name, bound-n)

The LBOUND function
Returns the lower bound in a specified dimension:

    LBOUND<n>(array-name)
    LBOUND(array-name, bound-n)


    DATA ex6_3(drop=i);
      SET sbp;
      ARRAY sbpary[*] _numeric_;
      DO i=1 to dim(dbpary);
        IF sbpary[i]=999 THEN sbpary[i]=.;
      END;
    RUN;

IN and OF Operator with an Array
Create a variable MISS, which is to indicate wheter SBP1-SBP6 contains missing values.


| SBP1 | SBP2 | SBP3 | SBP4 | SBP5 | SBP6 | MISS |
| ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| 136  | 140  | 137  | 117  | 116  | 124  | 0    |
| 142  | .    | 138  | 119  | 119  | 122  | 1    |
| .    | 141  | 139  | 119  | 120  | .    | 1    |
| 141  | 142  | 142  | 118  | 121  | 123  | 0    |

    DATA ex6_4;
      SET sbp2;
      ARRAY sbpary[*] _numeric_;
      MISS = . IN sbpary;
    RUN;

You can pass an array on to most functions with the OF operator.
Create two variables: SBP_MIN and SBP_MAX:


    DATA ex6_5;
      SET sbp2;
      ARRAY sbpary[*] _numeric_;
      sbp_min = min(of sbpary[*]);
      sbp_max = max(of sbpary[*]);
    RUN;
    
    TITLE 'Using the OF operator to create variables SBP_M and SBP_MAX'
    PROC_PRINT DATA=ex6_5;
    RUN;

Creating a Group of Variables by Using Arrays


    DATA ex6_6 (drop=i);
      SET sbp2;
      ARRAY sbp[6];
      ARRAY above[6];
      ARRAY threshhold[6] _temporary_ (140 140 140 120 120 120);
      DO i=1 to 6;
        IF (not missing(sbp[i]))
            THEN above[i]=sbp[i]>threshold[i]
       END;
    RUN;

Calculating Products of Multiple Variables


    DATA ex6_7(drop=i);
      SET product;
      ARRAY num[4];
      IF missing(num[1]) THEN result = 1;
      ELSE result = num[1];
      DO i=2 to 4;
        IF not missing(num[i])
            THEN result = result*num[i];
        END;
    RUN;

TRANSPOSE WIDE TO LONG WITH ARRAY


    DATA ex6_8(drop=s1-s3);
      SET wide;
      ARRAY s[3];
      DO time=1 to 3;
      score=s[time];
      IF not missing(score)
        THEN OUTPUT;
      END;
    RUN;

TRANSPOSE LONG TO WIDE WITH ARRAY


    DATA WIDEMES;
      SET CIUDADEDUC;
      BY AREA;
      KEEP AREA EDUC1 - EDUC3;
      RETAIN EDUC1 - EDUC3;
      ARRAY edu[3] $20 EDUC1 - EDUC3;
      IF first.AREA THEN DO;
              DO i= 1 to 3;
                edu[i]='';
              END;
      END;
      edu[MES]=EDUC;
      IF last.AREA THEN OUTPUT;
    RUN;


COMBINING SAS DATA SETS VERTICALLY
| Method        | End Result       | Order of observations     | Number of Input Data Sets |
| ------------- | ---------------- | ------------------------- | ------------------------- |
| Appending     | One data set     | Same as original data set | Only two                  |
| Concatenating | One new data set | Same as original data set | two or more               |

APPEND


    PROC APPEND BASE =customers_data;
            DATA=customers_new;
    RUN;
    PROC PRINT DATA=customers_data; 
    RUN;

Situaciones que pueden ocurrir cuando se hace una append:

![](https://paper-attachments.dropbox.com/s_838766B75B7717404C528139F5BBCED8D6BCF674885F88AE209B072CC261F8BF_1654874056952_image.png)

![](https://paper-attachments.dropbox.com/s_838766B75B7717404C528139F5BBCED8D6BCF674885F88AE209B072CC261F8BF_1654874312834_image.png)



![](https://paper-attachments.dropbox.com/s_838766B75B7717404C528139F5BBCED8D6BCF674885F88AE209B072CC261F8BF_1654875249034_image.png)


With the FORCE option it will drop the variable email and keep the information of age and gender from the Data I’m appending.


DATA STEP - CONCATENATE 

Concatenating copies all the information from the first data set and copies the information from the additional data sets into an additional data set.


    DATA SAS-data-set;
      SET SAS-data-set1 SAS-data-set2...;
      <additional SAS statements>
    RUN;



    DATA advisees_MPH;
      INPUT first $ gender $ program $;
      DATALINES;
      ALISON F MPH
      Ming F MPH
      Brian M MPH
    RUN;
    
    DATA advisees_DrPH;
      INPUT first $ gender $ program $;
      DATALINES;
      Tyffany F DrPH
      Florence F DrPH
    RUN;
    
    DATA advisees;
      SET advisees_MPH advisees_DrPH;
    RUN;
    PROC PRINT DATA =advisees;
    RUN;
    
    /*Cuando las bases empiezan con el mismo nombre*/
    DATA advisees;
      SET advisees:;
    RUN;


| first    | gender | program |
| -------- | ------ | ------- |
| Brian    | M      | MPH     |
| Ming     | F      | MPH     |
| Alison   | F      | MPH     |
| Florence | D      | DrPH    |
| Tyffany  | F      | DrPH    |

¿Que hacer cuando hay una variable con diferente nombre en las bases?


    DATA advisees_MPH;
      INPUT first $ gender $ degree $;
      DATALINES;
      ALISON F MPH
      Ming F MPH
      Brian M MPH
    RUN;
    
    DATA advisees_DrPH;
      INPUT first $ gender $ program $;
      DATALINES;
      Tyffany F DrPH
      Florence F DrPH
    RUN;
    
    DATA advisees;
      SET advisees_MPH advisees_DrPH (rename = (degree = program) );
    RUN;
    PROC PRINT DATA =advisees;
    RUN;


INTERLEAVING

Interleaving combines two or more SAS data sets into a single data set by interspersing their rows with one another based on the values of common BY variables. In some programming languages, the term merge means to interleave rows. However, rows that are interleaved in SAS data sets are not merged; they are copied from the original data sets in the order of the values of the BY variable.


    DATA interleav;
      SET data1 data2; 
      BY year; 
    RUN;
SQL - CONCATENATE 
    proc sql;
    select * from data1
    union all corresponding
    select * from data2;
    quit;
MERGE

SQL: lazy option
DATA STEP: I have more control 

TYPE OF JOINS


![](https://www.sqlshack.com/wp-content/uploads/2021/08/sql-join-chart-custom-poster-size-sql.png)


Los siguientes son los más comunes:

- LEFT INCLUSIVE es conocido como LEFT JOIN
- RIGHT INCLUSIVE es conocido como RIGHT JOIN
- FULL OUTER INCLUSIVE es conocido como FULL JOIN
- INNER JOIN es conocido como INNER JOIN

BASE A

| PAIS              | X |
| ----------------- | - |
| Antigua y Barbuda | 8 |
| Argentina         | 2 |
| Barbados          | 3 |

BASE B

| PAIS      | Y  |
| --------- | -- |
| Argentina | 60 |
| Alemania  | 32 |
| Austria   | 40 |


INNER JOIN
Mantiene solo las personas que están en ambas bases de datos. 
Por ejemplo, tengo una base de países miembros de la OECD (Base A) y países miembros del BID (Base B), el INNER JOIN me mantendría solo la información de los países que están en ambos tratados internacionales.

LEFT JOIN
Prioriza la información de la Base A, y solo agrega la información de la Base B de los países que están también en A.
Por ejemplo: Antigua y Barbuda está en la OECD pero no está en el BID, este país se mantiene en la base final. Suiza está en el BID pero no está en la OECD, este país no se mantiene en la base final.

RIGHT JOIN
La misma lógica que con LEFT JOIN pero solo mantiene los países de la base A y la información de la base B de los países que están en la base A.

FULL JOIN
Une las dos bases sin borrar información de ninguna de las bases de datos. (Es el que hace STATA)


TABLE RELATIONSHIPS


![](https://paper-attachments.dropbox.com/s_838766B75B7717404C528139F5BBCED8D6BCF674885F88AE209B072CC261F8BF_1654181245844_image.png)



BASIC PROC SQL SYNTAX FULL JOIN
    TITLE "SQL Full Join Results"
    
    PROC SQL number;
    CREATE TABLE sql_join AS
    SELECT *
      FROM WORK.OECD FULL JOIN WORK.BID
      ON PAIS=NombrePais; 
    /*Pais es la variable de la OECD, NombrePais es la variable del BID*/
    QUIT;


| PAIS              | NombrePais | X | Y  |
| ----------------- | ---------- | - | -- |
| Antigua y Barbuda |            | 8 |    |
| Argentina         | Argentina  | 2 | 60 |
| Barbados          |            | 3 |    |
|                   | Alemania   |   | 32 |
|                   | Austria    |   | 40 |

BASIC DATA STEP SYNTAX FULL JOIN

Primero hay que organizar las bases de datos por la llave principal


    PROC SORT data=WORK.OECD
              out=OECD_sorted;
        BY PAIS;
    RUN;


    PROC SORT data=WORK.BID
              out=BID_sorted;
        BY NombrePais;
    RUN;


    DATA ds_merge;
        LENGTH NombrePais $17;
        MERGE OECD_sorted (RENAME=(PAIS=NombrePais))
              BID_sorted;
        BY NombrePais;
    RUN;
    
    Title "Data Step Full Merge Results"
    PROC PRINT DATA=ds_merge;
    RUN;

The results are the same as with the SQL procedure but there is only one column with the name of the country.


| NombrePais        | X | Y  |
| ----------------- | - | -- |
| Antigua y Barbuda | 8 |    |
| Argentina         | 2 | 60 |
| Barbados          | 3 |    |
| Alemania          |   | 32 |
| Austria           |   | 40 |


¿Cómo sé cuál es más rápido?
Poniendo la siguiente función al principio:


    OPTIONS FULLSTIMER;
    OPTIONS NOFULLSTIMER;

Y revisar las notas que produce el LOG

BASIC PROC SQL SYNTAX LEFT & RIGHT JOIN

LEFT


    OPTIONS FULLSTIMER;
    
    TITLE "SQL Left Join Results"
    
    PROC SQL number;
    CREATE TABLE sql_join AS
    SELECT COALESCE(Pais,NombrePais) AS NombrePais, X, Y
      FROM WORK.OECD LEFT JOIN WORK.BID
      ON PAIS=NombrePais; 
        ORDER BY NombrePais;
    QUIT;
    OPTIONS NOFULLSTIMER;

The COALESCE() function returns the first non-null value in a list. In this case, if Pais is NULL it will put the value in NombrePais, if Pais is non-null it will return the value in Pais.

| NombrePais        | X | Y  |
| ----------------- | - | -- |
| Antigua y Barbuda | 8 |    |
| Argentina         | 2 | 60 |
| Barbados          | 3 |    |

RIGHT

    OPTIONS FULLSTIMER;
    
    TITLE "SQL Left Join Results"
    
    PROC SQL number;
    CREATE TABLE sql_join AS
    SELECT COALESCE(Pais,NombrePais) AS NombrePais, X, Y
      FROM WORK.OECD RIGHT JOIN WORK.BID
      ON PAIS=NombrePais; 
        ORDER BY NombrePais;
    QUIT;
    
    OPTIONS NOFULLSTIMER;
| NombrePais | X | Y  |
| ---------- | - | -- |
| Argentina  | 2 | 60 |
| Alemania   |   | 32 |
| Austria    |   | 40 |

BASIC DATA STEP SYNTAX LEFT & RIGHT JOIN

LEFT


    PROC SORT data=WORK.OECD
              out=OECD_sorted;
        BY PAIS;
    RUN;


    PROC SORT data=WORK.BID
              out=BID_sorted;
        BY NombrePais;
    RUN;


    DATA ds_merge;
        RETAIN NombrePais, X, Y;
        KEEP NombrePais, X, Y;
        LENGTH NombrePais $17;
        MERGE OECD_sorted(IN=inA RENAME=(PAIS=NombrePais))
              BID_sorted(IN=inB);
        BY NombrePais;
        IF inA;
    RUN;
    
    Title "Data Step LEFT Merge Results"
    PROC PRINT DATA=ds_merge;
    RUN;

The IN= creates a temporary variable during execution time that I can use to say I only want these rows if they are in the first table or if they are in the second table. InA and InB are just names I selected but I can call my temporary variables anyway I want.
Then I use the temporary variable in a subsetting statement “IF” .
With the option KEEP I specify the variables I want to keep in my output data.
With the option RETAIN I specify the order I want those variables to appear in my output data.

RIGHT


    PROC SORT data=WORK.OECD
              out=OECD_sorted;
        BY PAIS;
    RUN;


    PROC SORT data=WORK.BID
              out=BID_sorted;
        BY NombrePais;
    RUN;


    DATA ds_merge;
        RETAIN NombrePais, X, Y;
        KEEP NombrePais, X, Y;
        LENGTH NombrePais $17;
        MERGE OECD_sorted(IN=inA RENAME=(PAIS=NombrePais))
              BID_sorted(IN=inB);
        BY NombrePais;
        IF inB;
    RUN;
    
    Title "Data Step LEFT Merge Results"
    PROC PRINT DATA=ds_merge;
    RUN;

BENEFITS OF THE DATA STEP
Full control over the whole processing
You can create multiple tables

    DATA OECD_AND_BID OECD_NO_BID BID_NO_OECD;
        RETAIN NombrePais, X, Y;
        KEEP NombrePais, X, Y;
        LENGTH NombrePais $17;
        MERGE OECD_sorted(IN=inA RENAME=(PAIS=NombrePais))
              BID_sorted(IN=inB);
        BY NombrePais;
        IF inA AND inB THEN OUTPUT OECD_AND_BID;
        ELSE IF inA AND NOT inB THEN OUTPUT OECD_NO_BID;
        ELSE OUTPUT BID_NO_OECD;
    RUN;
    
    Title "Paises en la OECD y miembros del BID"
    PROC PRINT DATA=OECD_AND_BID;
    RUN;
    
    Title "Paises en la OECD que no son miembros del BID"
    PROC PRINT DATA=OECD_NO_BID;
    RUN;
    
    Title "Paises miembros del BID que no hacen parte de la OECD"
    PROC PRINT DATA=BID_NO_OECD;
    RUN;

If i click Debug I can see the Data Step process while it is happening.

BENEFITS OF THE SQL JOIN
Joining multiple tables
Non-equijoin

TableC

| Country_starts | Country_ends | Z  |
| -------------- | ------------ | -- |
| A              | A            | 58 |
| B              | S            | 36 |




    TITLE "Joining multiple tables"
    TITLE2 "with different criteria"
    
    PROC SQL number;
    CREATE TABLE sql_join AS
    SELECT COALESCE(Pais,NombrePais) AS NombrePais, X, Y, Z
      FROM WORK.OECD LEFT JOIN WORK.BID
      ON PAIS=NombrePais;
              FULL JOIN WORK.TableC
              ON Country_starts<=NombrePais<Country_ends; /*This is a non-equijoin*/
        ORDER BY NombrePais;
    QUIT;


| NombrePais        | X | Y  | Z  |
| ----------------- | - | -- | -- |
| Antigua y Barbuda | 8 |    | 36 |
| Argentina         | 2 | 60 |    |
| Barbados          | 3 |    | 36 |

UPDATE

Creates a new, updated data set by using rows from a transaction data set to change the values of corresponding rows in a master data set. The master data set contains the original information, and the transaction data set contains the new information that needs to be applied to the master data set.

    DATA Updated;
      UPDATE Master Transaction;
      BY Year;
    RUN;


![](https://paper-attachments.dropbox.com/s_838766B75B7717404C528139F5BBCED8D6BCF674885F88AE209B072CC261F8BF_1654885346480_image.png)



MODIFY

Matches the values of one or more BY variables in a master data set against the same variables in a transaction data set. The existing master data set is modified. A new data set is not created.


    DATA Master;
      MODIFY Master Transaction;
      BY Year;
    RUN;
![](https://paper-attachments.dropbox.com/s_838766B75B7717404C528139F5BBCED8D6BCF674885F88AE209B072CC261F8BF_1654885457079_image.png)



MACRO LANGUAGE
- A macro variable stores a text string that can be substituted into a SAS program.
- The process to run a SAS program is: scan parses statements into tokens, tokens are sent to compiler for syntax checking, execution occurs when step boundary is reached.
- Macro variables can be referenced in a program by preceding the macro variable name with an &.
- If a macro variable reference is used inside quotation marks, double quotation marks must be used.
- Cuando hay una macro % o & primero corre los macro y crea un output con esas macro, luego rescan para ver si hay otra macro y así sucesivamente hasta que no hay macros y puede ejecutar.
- OPTIONS MLOGIC MPRINT SYMBOLGEN (Muestran el proceso de la macro en el log, sirve para ver y entender qué está haciendo la macro). MLOGIC muestra en qué paso esta, SYMBOLGEN muestra que argumento toma la macro y MPRINT la línea del comando que acaba de correr.


![](https://paper-attachments.dropbox.com/s_838766B75B7717404C528139F5BBCED8D6BCF674885F88AE209B072CC261F8BF_1654630189961_image.png)



LET

- The %LET statement defines the macro variable name and assigns a value, corre al momento que se ejecutan los macro references, antes de correr el data step.
- Para poner  una macro que depende de otra macro uso dos veces && y tres veces si el valor de la macro es lo que quiero llamar como macro. Multiple & force the macro processor to scan the macro variable reference more than once.

Por ejemplo:

    %LET year=2020;
    %LET var=location;
    %LET city2020=Washington;
    %LET location2020=District of Columbia
    %PUT &city&2020; ERROR
    %PUT &&city&2020; 🙂 Washington
    %PUT &&&var&2020; 🙂 District of Columbia

CALL SYMPUTX
It executes at data step execution time, it is not actually a macro language statement, it is a data step statement, it puts a value in the symbol table, which is where macro variables are stored, but it is not part of the macro language itself.

- The limitation of LET is that macro processor assigns value before SAS code executes, to create a macro variable at the execution time using DATA step I can use CALL SYMPUTX 
    DATA _NULL_;
      SET sashelp.class;
        WHERE name='Alfred';
        %LET alfred_age=age; 
    /*Asigna la palabra literal 'age' porque primero lee las macros y luego el datastep*/
    RUN;
    DATA _NULL_;
      SET sashelp.class;
        WHERE name='Alfred';
        CALL SYMPUTX("alfred_age", age);
    RUN;

Creating macro variables at execution time using PROC SQL

    PROC SQL NOPRINT;
      SELECT age INTO: alfred_age TRIMMED
      FROM sashelp.class 
      WHERE name='Alfred';
    QUIT;
    
    OPTIONS NOSOURCE;
    %PUT ============================;
    %PUT Alfred age is %alfred_age..;
    %PUT ============================;
    OPTIONS SOURCE;

Horizontal vs Vertical Macro Variable Lists
Horizontal list: a list of values in a single macro variable

    %LET origin_list=Asian Europe USA;
    %LET origin_list=Asian,Europe,USA;

With Data Step

    DATA _NULL_;
      SET sashelp.cars end=eof;
      LENGTH origin_list $200;
      RETAIN origin_list;
       origin_list=CATX('/', origin_list, origin);
      IF eof THEN DO;
        CALL symputx('origin_list', origin_list);
        CALL symputx('numorigin', _n_);
      END;
    RUN;
    
    %PUT &origin_list;
    %PUT Hay &numorigin paises;

With PROC SQL

    /*Horizontal*/
    PROC SQL NOPRINT;
      SELECT DISTINC origin
        INTO: origin_list separated by '/'
        FROM sashelp.cars
      ORDER BY origin;
      %LET numorigins = &sqlobs;
    QUIT;

Using Horizontal Macro Variable Lists

    %SCAN(&origin_list, 1,/) /*-> resolves to Asia*/
    %SCAN(&origin_list, 2,/) /*-> resolves to Asia*/
    %SCAN(&origin_list, 3,/) /*-> resolves to Asia*/

Use loop counter as index for %SCAN function:

    %MACRO origenes;
    %DO i = 1 %TO &numorigins;
      %PUT Item &i: %SCAN(&origin_list, &i, /);
    %END;
    %MEND origenes;


    %MACRO graph_stocks;
      PROC SQL NOPRINT;
                    SELECT DISTINCT stock INTO: STOCK_LIST separated by '~'
                            FROM sashelp.stocks;
                    %LET NUM_STOCKS=&sqlobs;
            QUIT;      
      %DO I = 1 %TO &NUM_STOCKS;
      %LET stonks= %SCAN(&STOCK_LIST, &I, ~);
        ods pdf file = "&stonks..pdf";
          PROC SGPLOT DATA=sashelp.stocks;
            WHERE stock = "&stonks";
            HIGHLOW x=date high=high low=low;
          RUN;
        ods pdf close;
      %END;
    %MEND graph_stocks;
    %graph_stocks;

Vertical list: separate macro variable for each item

    %LET origin1 = Asia;
    %LET origin2 = Europe;
    %LET origin3 = USA;


    PROC SORT DATA=sashelp.cars 
      OUT=unique_origins(keep=origin) NODUPKEY;
    BY origin;
    
    DATA _NULL_;
      SET unique_origins END=eof;
      CALL SYMPUTX(CATS('origin', _n_),origin);
      IF eof THEN CALL SYMPUTX('numorigins',_n_);
    RUN;

With PROC SQL

    /*Vertical*/
    PROC SQL NOPRINT;
      SELECT DISTINC origin
        INTO: origin1-
        FROM sashelp.cars
      ORDER BY origin;
      %LET numorigins = &sqlobs;
    QUIT;


    %MACRO split_data;
      PROC SQL NOPRINT;
        SELECT DISTINCT origin
          INTO: ORIGIN1-
          FROM sashelp.cars;
        %LET NUM_ORIGINS = &sqlobs;
      QUIT;
      
      %DO I = 1 %TO &NUM_ORIGINS;
          DATA cars_&&ORIGIN&I;
          SET sashelp.cars;
            WHERE origin = "&&ORIGIN&I";
          RUN;
      %END;
    %MEND split_data;
    
    OPTIONS MPRINT;
    %split_data;


- Puedo poner valores default para los argumentos en las macro. Ejemplo:
    %MACRO test(x=0);
      %IF &x. > 0 %THEN %DO;
        proc means data = sashelp.class MEAN;
          var age;
        RUN;
      %END;
      %ELSE %PUT el argumento tiene que ser mayor a &x;
    %MEND;
    %test;


- STR y NSTR ayudan a poder tener macrovariables que tienen palabras como if, . ;”, do, entre otras. NSTR ayuda a incluir & y % en las macro y trata estos caracteres especiales como texto.
    %LET step= data new%STR(;) x=1%STR(;);
    %LET title= Employee%STR(') Report;
    %LET step=%STR(o . ^ i);
    %LET title= R%NSTR(&)D as of &sysdate9;
    %LET title= NSTR(%) of sells;


- SUPERQ me ayuda a lidiar con esto sin necesidad de marcar cada caracter individualmente
    %MACRO check(company);
      %IF ABC NE %SUPERQ(company) %THEN %DO;
        %PUT Not a match;
      %END;
      %ELSE %put Match;
    %MEND check;
    
    %check(OR Insurance);
    
    DATA _NULL_;
      CALL SYMPUT('company','Smith&Jones');
    RUN;
    %LET newcmpy=%SUPERQ(company);
    %PUT The new company name is &newcmpy;


- MINOPETAROR me permite hacer una operación lógica en la macro contra un valor que tenga espacios o algún delimitador, el default es espacios, pero se puede cambiar con MINDELIMITER
    %MACRO check(company) / MINOPETAROR MINDELIMITER=',';
      %IF &company IN A,B,C %THEN %DO;
        %PUT Not a match;
      %END;
      %ELSE %put Match;
    %MEND check;
    
    %check('A,B,C');


- %RETURN statement causes normal termination of the currently executing macro
- %GOTO statement branches macro processing to the specified label (the spot where you want execution to branch to).
    %LET posclass=%SYSFUNC(VARNUM(&dsid,&class));
      %IF &posclass=0 %THEN %GOTO invalid;
    %LET posvar=%SYNFUNC(VARNUM(&dsid, &var));
      %IF &posvar=0 %THEN %GOTO invalid;
    %LET vart=%SYNFUNC(VARTYPE(&dsid, &var));
      %IF &vart=C %THEN %DO;
        %invalid: %PUT ERROR: Variable is not valid.;
        %LET dsid=%SYSFUNC(CLOSE(dsid));
        %RETURN;
    %END;

When to use IF and %IF in SAS Macros
IF statement cannot be used outside data step whereas %IF can be used outside and inside data step but within the macro.
The %IF can only be used inside of a macro. In other words, you cannot use it in a program without it in a macro definition (%macro yourmacro; %mend;)
SAS Macro will be executed first and once completed, data step statements will be executed.


    %macro temp(N=);
      %if &N. > 10 %then %do;
        proc means data = sashelp.class MEAN;
          var age;
        run;
      %end;
      %else %put better luck next time;
    %mend;
    %temp(N=19);


![](https://paper-attachments.dropbox.com/s_838766B75B7717404C528139F5BBCED8D6BCF674885F88AE209B072CC261F8BF_1654524296406_image.png)


Example of using both %IF and regular IF in the same step:

    %macro mtest(type=yes); 
      data test; 
        %if type=yes %then %do; 
          keep chol_status sex weight height total; 
        %end; 
        %else %do; 
          keep sex weight height; %end; set patient_info; 
          if chol_status ne ' ' then Total=LDL+HDL; 
      Run; 
    %mend;

When either of DO END or %DO %END can be used
If you're generating code to automate repetitive task within a data step, and you can use either %do-%end or do-end.

| Method I : DO END                                                                                                        | Method II : %DO %END                                                                                                                                                                              |
| ------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| DATA temp;<br>    DO j =1 TO 5;<br>            N = j *5;<br>            OUTPUT;<br>       END;<br>       DROP j;<br>RUN; | %MACRO test;<br>       DATA temp;<br>              %DO j = 1 %TO 5;<br>                     N = &j. *5;<br>                     OUTPUT;<br>              %END;<br>       RUN;<br>%MEND;<br>%test; |

SAS I/O Functions
SAS provides I /O functions that can be used to retrieve metadata information about a data set.
EXIST: revisa si una base existe
OPEN/CLOSE: toma el valor de 1 si esta abierta/cerrada y 0 si no
ATTRC/ATTRN: da informacion de atributos como numero de observaciones o variables
VARLABEL/VARLEN/VARTYPE: da informacion sobre las variables, para el tipo de variables C es character y N numero

EXTERNAL FILES FUNCTIONS
They are similar to SAS I/O functions except the external file functions return information that is associated with external files.
DOPEN/DCLOSE: opens and closes a directory
DNUM: number of members in a directory
DREAD: name of a directory member

CALL EXECUTE

Call Execute is a facility of the DATA step which allows executing SAS code generated by the DATA step. Also, the data from the DATA step can be used as part of the executable code in the Call Execute.


    DATA _NULL_; 
      CALL EXECUTE('%PUT Hello world;'); 
    RUN;

Call Execute becomes especially powerful when you start including values from a dataset in the arguments.


    DATA _NULL_; 
      set sashelp.class; 
      call execute('%put Students name is '||strip(name)||'; '); 
    RUN;

It is also very powerful to run 

Analyzing and Reporting on Data
----------

Enhancing Reports with Titles, Footnotes, and Labels

- TITLE is a global statement that establishes a permanent title for all reports created in your SAS session.
- You can have up to 10 titles, in which case you would just use a number 1-10 after the keyword TITLE to indicate the line number. TITLE and TITLE1 are equivalent.
- Titles can be replaced with an additional TITLE statement with the same number. TITLE; clears all titles.
- You can also add footnotes to any report with the FOOTNOTE statement. The same rules for titles apply for footnotes.
- Labels can be used to provide more descriptive column headers. A label can include any text up to 256 characters.
- All procedures automatically display labels with the exception of PROC PRINT. You must add the LABEL option in the PROC PRINT statement.
| TITLE<n> "title-text"; |

| FOOTNOTE<n> "footnote-text"; |

| LABEL col-name="label-text"; |

- To create a grouped report, first use PROC SORT to arrange the data by the grouping variable, and then use the BY statement in the reporting procedure.
| PROC procedure-name;<br>        BY col-name;<br>RUN; |

Creating Frequency Reports

- PROC FREQ creates a frequency table for each variable in the input table by default. You can limit the variables analyzed by using the TABLES statement.
| PROC FREQ DATA=input-table;<br>        TABLES col-name(s) < / options>;<br>RUN; |

- PROC FREQ statement options:
- ORDER=FREQ|FORMATTED|DATA
- NLEVELS
- TABLES statement options:
- NOCUM
- NOPERCENT
- PLOT=FREQPLOT (must turn on ODS graphics)
- OUT=output-table
- One or more TABLES statements can be used to define frequency tables and options.
- ODS graphics enable graph options to be used in the TABLES statement.
- WHERE, FORMAT, and LABEL statements can be used in PROC FREQ to customize the report.
- When you place an asterisk between two columns in the TABLES statement, PROC FREQ produces a two-way frequency or crosstabulation report.
| PROC FREQ DATA=input-table;<br>        TABLES col-name*col-name < / options>;<br>RUN; |

Creating Summary Statistics Reports

- Options in the PROC MEANS statement control the statistics included in the report.
- The CLASS statement specifies variables to group the data before calculating statistics.
- The WAYS statement specifies the number of ways to make unique combinations of class variables.
| PROC MEANS DATA=input-table <stat-list> < / options>;<br>        VAR col-name(s);<br>        CLASS col-name(s);<br>        WAYS n;<br>RUN; |

- The OUTPUT statement creates an output SAS table with summary statistics. Options in the OUTPUT statement determine the contents of the table.
| PROC MEANS DATA=input-table <stat-list> < / options>;<br>        VAR col-name(s);<br>        CLASS col-name(s);<br>        WAYS n;<br>        OUTPUT OUT=output-table <statistic(col-name)=col-name> < / option(s);<br>RUN; |

Exporting Reports

- The SAS Output Delivery System (ODS) is programmable and flexible, making it simple to automate the entire process of exporting reports to other formats.
| ODS <destination> <destination-specifications>;<br><br>/* SAS code that produces output */<br><br>ODS <destination> CLOSE; |

- Selected destinations:
    - CSVALL
    - PowerPoint
    - RTF
    - PDF
- Exporting results to Excel:
| ODS EXCEL FILE="filename.xlsx" STYLE=style<br>                      OPTIONS(SHEET_NAME='label');<br><br>/* SAS code that produces output */<br><br>ODS EXCEL OPTIONS(SHEET_NAME='label');<br><br>/* SAS code that produces output */<br><br>ODS EXCEL CLOSE; |

- The ODS EXCEL destination creates an .XLSX file.
- By default, each procedure output is written to a separate worksheet with a default worksheet name. The default style is also applied.
- Use the STYLE= option on the ODS EXCEL statement to apply a different style.
- Use the OPTIONS(SHEET_NAME=’label’) on the ODS EXCEL statement to provide a custom label for each worksheet.
- Exporting results to PDF:
| ODS PDF FILE="filename.pdf" STYLE=style<br>                 STARTPAGE=NO PDFTOC=1;<br>       <br>ODS PROCLABEL"label";<br>/* SAS code that produces output */<br><br>ODS PDF CLOSE; |

- The ODS PDF destination creates a .PDF file.
- The PDFTOC=n option controls the level of the expansion of the table of contents in PDF documents.
- The ODS PROCLABEL statement enables you to change a procedure label.


Using the Hash and Hash Iterator Objects

To study more…

Hash Table Merging

The DATA step hash object is a temporary, in-memory data structure that enables you to efficiently store, search, and retrieve data based on lookup keys. Hash objects are only available for the duration of the DATA step. Hash objects are initialized through the DECLARE statement. Methods such as FIND, ADD, and OUTPUT operate on hash objects. The contents of the hash object can be written to a data table by using the hash OUTPUT method. In-memory processing with hash tables is usually faster than other DATA step methods.


    declare hash h(dataset:'data1');
    rc = h.definekey('key');
    rc = h.definedata('data1');
    h.definedone();
    set data2;
    h.find();
    run;

https://go.documentation.sas.com/doc/en/pgmsascdc/9.4_3.5/lecompobjref/n1eufww2rt03gcn1spj5siri07c9.htm


DS2 Language

To study…
https://go.documentation.sas.com/doc/en/pgmsascdc/9.4_3.5/ds2ref/titlepage.htm


https://www.youtube.com/watch?v=HP7tzXHxkt0&


https://youtu.be/HP7tzXHxkt0



Código Vanessa

data _null_;
do i = 1 to 8;
call execute(cats('%nrstr(%GENERATE_COLLAPSE(num=',i,'));'));
end;
run;

proc delete data=work.Orbis_merge;
run;
