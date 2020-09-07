
<!-- README.md is generated from README.Rmd. Please edit that file -->

openfda-extract
===============

<!-- badges: start -->
<!-- badges: end -->

This repository collects [adverse events
data](https://open.fda.gov/apis/device/event/download/) from openFDA. A
single [script](/data-raw/loop.R) attempts to:

1.  Convert JSON files to tabular format using `jsonlite::fromJSON` and
    `tidyr::unnest`.
2.  Save the data to a database.

The data is relational as described
[here](https://opendata.stackexchange.com/a/2187). Converting the data
to tabular format may not be efficient and causes lots of duplication.
To avoid duplication, I have tried to store nested data in different
tables:

1.  `adverse_events`
2.  `adverse_events.mdr_text`
3.  `adverse_events.product_problems`
4.  `adverse_events.source_type`
5.  `adverse_events.device`
6.  `adverse_events.patient`
7.  `adverse_events.remedial_action`
8.  `adverse_events.type_of_report`

Where the naming convention is `mainframe.<list col>`.

Hardware
--------

1.  I transformed the data on:
    -   2013 15" MacBook Pro, 8 GB Memory, 8 Core CPU.
2.  I wrote the data to:
    -   Postgres database hosted on a digital ocean droplet
    -   2 GB Memory, 2 vCPUs, 60 GB Disk, Ubuntu 18.04.3 (LTS) x64

For context, this was the result of my first run:

    ~ openFDA database refresh completed in [17.4161201466454 hours]

Examples
--------

If you have successfully ran everything, you should have 8 tables with
millions of observations that you can explore.

    library(dplyr, warn.conflicts = FALSE)
    library(DBI)

    # credentials
    dw <- config::get("datawarehouse")

    # connect to db
    con <- DBI::dbConnect(
      odbc::odbc(),
      Driver = dw$driver,
      Server = dw$server,
      Database = dw$database,
      UID = dw$uid,
      PWD = dw$pwd,
      Port = dw$port
    )

    # list all available tables
    dbListTables(con)
    #> [1] "adverse_events"                  "adverse_events.device"          
    #> [3] "adverse_events.mdr_text"         "adverse_events.patient"         
    #> [5] "adverse_events.product_problems" "adverse_events.remedial_action" 
    #> [7] "adverse_events.source_type"      "adverse_events.type_of_report"

    # query the mdr text
    tbl(con, "adverse_events.mdr_text") %>% 
      select(id, text)
    #> # Source:   lazy query [?? x 2]
    #> # Database: postgres [tyler@localhost:/openfda]
    #>    id                 text                                                      
    #>    <chr>              <chr>                                                     
    #>  1 2019q4-0004-0004-… IT WAS REPORTED THAT A LOW TRANSMITTER BATTERY ALERT OCCU…
    #>  2 2019q4-0004-0004-… (B)(4).                                                   
    #>  3 2019q4-0004-0004-… IT WAS REPORTED THAT A LOW TRANSMITTER BATTERY ALERT OCCU…
    #>  4 2019q4-0004-0004-… (B)(4). CURRENTLY IT IS UNKNOWN WHETHER OR NOT THE DEVICE…
    #>  5 2019q4-0004-0004-… THE CUSTOMER REPORTED VIA PHONE CALL THAT RETAINER RING C…
    #>  6 2019q4-0004-0004-… THE INSULIN PUMP WAS RECEIVED WITH MISSING RETAINER AND M…
    #>  7 2019q4-0004-0004-… (B)(4). CURRENTLY IT IS UNKNOWN WHETHER OR NOT THE DEVICE…
    #>  8 2019q4-0004-0004-… CUSTOMER REPORTED VIA PHONE CALL THAT THE INSULIN PUMP HA…
    #>  9 2019q4-0004-0004-… THE INSULIN PUMP WAS RECEIVED WITH A CRITICAL PUMP ERROR …
    #> 10 2019q4-0004-0004-… (B)(4).                                                   
    #> # … with more rows

    # query the device information
    tbl(con, "adverse_events.device") %>% 
      select(id, manufacturer_d_name)
    #> # Source:   lazy query [?? x 2]
    #> # Database: postgres [tyler@localhost:/openfda]
    #>    id                     manufacturer_d_name                 
    #>    <chr>                  <chr>                               
    #>  1 2019q4-0004-0004-72690 MEDTRONIC PUERTO RICO OPERATIONS CO.
    #>  2 2019q4-0004-0004-72691 MEDTRONIC PUERTO RICO OPERATIONS CO.
    #>  3 2019q4-0004-0004-72692 MEDTRONIC PUERTO RICO OPERATIONS CO.
    #>  4 2019q4-0004-0004-72693 MEDTRONIC, INC.                     
    #>  5 2019q4-0004-0004-72694 ALLERGAN (COSTA RICA)               
    #>  6 2019q4-0004-0004-72695 MEDTRONIC PUERTO RICO OPERATIONS CO.
    #>  7 2019q4-0004-0004-72696 MEDTRONIC PUERTO RICO OPERATIONS CO.
    #>  8 2019q4-0004-0004-72697 MEDTRONIC PUERTO RICO OPERATIONS CO.
    #>  9 2019q4-0004-0004-72698 MEDTRONIC PUERTO RICO OPERATIONS CO.
    #> 10 2019q4-0004-0004-72699 MEDTRONIC PUERTO RICO OPERATIONS CO.
    #> # … with more rows

    # disconnect
    dbDisconnect(con)

Note that there is an `id` column in every table, for example:

-   `2014q4-0002-0003-3683`
-   `<year quarter>-<part n>-<n parts>-<row number>`

I made this column so that tables can be joined (though this ID in some
other form might already exist in the data, I just haven’t figured out
if that is the case or not).
