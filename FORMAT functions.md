DB2 ships with two very useful formatting utilities called **VARCHAR_FORMAT** and **TIMESTAMP_FORMAT**.

If you have ever dealt with tables that stored date and time in numeric format, you would be
aware of the tedious code that you would have to write in order to convert these into DATE and
TIME format that SQL can recognize.

Let's assume that ORIG_DATE and ORIG_TIME are date and time columns in a table and hold the following values -

* ORIG_DATE = 20180212
* ORIG_TIME = 000230

The following code will convert ORIG_DATE into a proper DATE format -
```SQL
date( substr(ORIG_DATE, 1, 4) || '-' ||
      substr(ORIG_DATE, 5, 2) || '-' ||
      substr(ORIG_DATE, 7, 2)
    ) as NEW_DATE
```

And the following code will convert ORIG_TIME into a TIME format -
```SQL
time( substr(ORIG_TIME, 1, 2) || ':' ||
      substr(ORIG_TIME, 3, 2) || ':' ||
      substr(ORIG_TIME, 5, 2)
    ) as NEW_TIME
```

And now if you combine the above two derived columns, NEW_DATE and NEW_TIME, you can
create a timestamp column as follows:
```SQL
  TIMESTAMP( NEW_DATE, NEW_TIME)
```
# TIMESTAMP_FORMAT
Here is where the first FORMAT function TIMESTAMP_FORMAT comes into play.
Using the function, all of the above code can be reduced to just one line as shown below -
```SQL
TIMESTAMP_FORMAT(ORIG_DATE ||' '|| ORIG_TIME, 'YYYYMMDD HH24MISS')
```

What the above expression does is look at the value in argument1 and convert it
into a timestamp format. Argument2 helps the function understand the format of the data in argument1.
This is again shown below -
### Understanding the function
```
TIMESTAMP_FORMAT(INPUT_VALUE, FORMAT_OF_INPUT_VALUE) ==> OUTPUT as TIMESTAMP
```
In the example referenced, the format was specified as YYYYMMDD HH24MISS.
The first part (YYYYMMDD) should be obvious. The latter part is explained below -
```
HH24 = Hours in 24 hour format
  MI = minutes
  SS = seconds
```
The function can be understood better by looking at variations of the original example as shown below-
```SQL
TIMESTAMP_FORMAT(ORIG_DATE ||'/'|| ORIG_TIME, 'YYYYMMDD/HH24MISS')

TIMESTAMP_FORMAT(ORIG_DATE || ORIG_TIME, 'YYYYMMDDHH24MISS')

TIMESTAMP_FORMAT(ORIG_DATE ||':'|| ORIG_TIME, 'YYYYMMDD:HH24MISS')
```

### Gotcha
Now, wasn't that easy? Not so fast! The above examples will fail when ORIG_TIME has leading zeroes.
The current value in ORIG_TIME (000230 or 00:02:30) will have its leading zeroes truncated and
the time component of the input will no longer match the formatting specified.

The way to fix the truncation of leading zeroes is by adding them back via the LPAD function.
```SQL
LPAD(ORIG_TIME, 6, '0')
```

Now, when we incorporate the above fix, the code will work for all scenarios.
```SQL
TIMESTAMP_FORMAT(ORIG_DATE ||'/'|| LPAD(ORIG_TIME, 6, '0'), 'YYYYMMDD/HH24MISS')
```
### #Slow_typer #dont_wanna_type_so_much
The function TIMESTAMP_FORMAT can also be referred to as TO_DATE. So the above example can also be re-written as follows-
```SQL
TO_DATE(ORIG_DATE ||'/'|| LPAD(ORIG_TIME, 6, '0'), 'YYYYMMDD/HH24MISS') as PERIOD_IN_TS
```

# VARCHAR_FORMAT
Now that we have converted our data into a proper timestamp format, we can use it with **VARCHAR_FORMAT**
to format any which way we want.
```SQL
VARCHAR_FORMAT(PERIOD_IN_TS, 'Mon Day YYYY-MM-DD HH24:MI') => Feb Saturday 2018-02-24 00:02
VARCHAR_FORMAT(PERIOD_IN_TS, 'Mon Dy YYYY-MM-DD HH24:MI') => Feb Sat 2018-02-24 00:02
VARCHAR_FORMAT(PERIOD_IN_TS, 'Month YYYY-MM-DD HH24:MI') => February 2018-02-24 00:02
VARCHAR_FORMAT(PERIOD_IN_TS, 'Q YYYY-MM-DD HH:MI PM') => 1 2018-02-24 12:02 AM
```
### Understanding the function
`VARCHAR_FORMAT(INPUT_VALUE, DESIRED_OUTPUT_FORMAT) => Formatted Output`

### Crazy formatting
```SQL
VARCHAR_FORMAT(PERIOD_IN_TS, ''YYYY;;;;/.MM;;DD HH:MI PM'') => 2018;;;;/.02;;24 12:02 AM
```
### #Slow_typer #dont_wanna_type_so_much
The function VARCHAR_FORMAT can also be referred to as TO_CHAR. This allows the above example to be re-written as -
```SQL
TO_CHAR(PERIOD_IN_TS, 'Month Day YYYY-MM-DD HH24:MI')
```
## References
* [VARCHAR_FORMAT documentation](https://www.ibm.com/support/knowledgecenter/en/SSEPGG_9.7.0/com.ibm.db2.luw.sql.ref.doc/doc/r0007110.html)
* [TIMESTAMP_FORMAT documentation](https://www.ibm.com/support/knowledgecenter/en/SSEPGG_9.7.0/com.ibm.db2.luw.sql.ref.doc/doc/r0007107.html)
