# sba-loans-covid-19-datasette

Notes on how I created https://sba-loans-covid-19.datasettes.com/

Twitter thread: https://twitter.com/simonw/status/1280283053726691329

## Data source

I'm using the `150k plus/foia_150k_plus.csv` file from the zip archive released here: https://sba.app.box.com/s/wz72fqag1nd99kj3t9xlq49deoop6gzf

Accompanying press release: https://home.treasury.gov/news/press-releases/sm1052

More downloads (e.g. per-state): https://home.treasury.gov/policy-issues/cares-act/assistance-for-small-businesses/sba-paycheck-protection-program-loan-level-data

## Converting to SQLite

I used [csvs-to-sqlite](https://github.com/simonw/csvs-to-sqlite) to convert the file to SQLite and make it searchable like this:

    csvs-to-sqlite 150k\ plus/foia_150k_plus.csv \
        -t foia_150k_plus \
        /tmp/better-loans.db \
        -c LoanRange \
        -c City \
        -c State \
        -c BusinessType \
        -c RaceEthnicity \
        -c Gender \
        -c Veteran \
        -c NonProfit \
        -d DateApproved \
        -c Lender \
        -c CD \
        -f BusinessName \
        -f Address

## Adding NAICS codes

The `NAICSCode` column contains six digit NAICS codes, which correspond to different industries.

I downloaded the "6-digit 2017 Code File" XLS file from https://www.census.gov/eos/www/naics/downloadables/downloadables.html and opened it in Numbers, then exported the data back out again as a two column CSV: [naics_2017.csv](https://github.com/simonw/sba-loans-covid-19-datasette/blob/main/naics_2017.csv)

Then I ran the following commands - using [sqlite-utils](https://sqlite-utils.readthedocs.io/en/stable/cli.html) - to import that data and set it up as a foreign key from the `NAICSCode` column.

    # First create the table with an integer primary key and a text column
    sqlite-utils create-table loans_150k_plus.db naics_2017 id integer name text --pk=id
    # Now import the data into it
    sqlite-utils insert loans_150k_plus.db naics_2017 naics_2017.csv --csv
    # Configure the foreign key
    sqlite-utils add-foreign-key loans_150k_plus.db foia_150k_plus NAICSCode naics_2017 id
    # Add an index to that column
    sqlite-utils create-index loans_150k_plus.db foia_150k_plus NAICSCode

## Publishing to Cloud Run

I published the database by running the following command:

    datasette publish cloudrun \
        loans_150k_plus.db \
        --service sba-loans \
        --source=SBA \
        --source_url=https://sba.app.box.com/s/tvb0v5i57oa8gc6b5dcm9cyw7y2ms6pp \
        --title="COVID-19 SBA loans above 150k" \
        --memory 2Gi \
        --about=sba-loans-covid-19-datasette \
        --about_url=https://github.com/simonw/sba-loans-covid-19-datasette
