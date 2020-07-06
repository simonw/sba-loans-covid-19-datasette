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
