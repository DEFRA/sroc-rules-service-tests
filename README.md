# SROC Rules Service Batch Tests

This project contains the acceptance tests in the form of batch tests for the Rules Service built using Postman which can be run using [Newman](https://learning.postman.com/docs/running-collections/using-newman-cli/command-line-integration-with-newman/). 

## Pre-Requisites 

You will need [Node.js](https://nodejs.org/en/) installed (ideally an LTS version)

You'll also need [Newman](https://learning.postman.com/docs/running-collections/using-newman-cli/command-line-integration-with-newman/) installed 

## Installation

All you need to do is clone the repository and drop into it

```bash
git clone https://github.com/DEFRA/sroc-rules-service-tests.git && cd sroc-rules-service-tests
```

## Configuration

> Important! Do not add environment files to source control

We have 2 environments where the Rules Service Tests could be running; **non-prod** and **prod**.

For each environment you wish to test you'll need to create an [environment file](https://learning.postman.com/docs/sending-requests/managing-environments/) in  `environments/`. An [example](/environments/example.postman_environment.json) with dummy data is provided as a reference.

For example, if you wanted to start testing the **non-prod** environment the steps would be

- duplicate [example.json](/environments/example.json)
- rename to something meaningful; `nonprod.json`
- update the `name` attribute to something meaningful: `"name": "Rules Service Non-Prod tests",`
- update the `value` attribute for each of the properties (`baseUrl`, `userName` & `password`) to match the environment

You'll need to contact an existing [team member](https://github.com/DEFRA/sroc-service-team) to obtain the proper credentials.

Git is setup to ignore everything bar the example environment file. Even so, double check your environment file has not been comitted before pushing it to GitHub.

## Test Structure

In order to adequately test the rules service, we opted for a data driven approach and made use of Postman's test runner and its ability to upload sets of data which allows us to test multiple permutations as individual requests. This is what we call a "batch" test. These batch tests are made up of 2 parts;

- _**An excel file**_ (found under Batch Work Sheets) that calculates expected charge values for any combination of rules service input data for a specific financial year 
- _**A corresponding Postman collection**_ which runs a csv version of the `Inputs` tab in the aforementioned excel file against the rules service financial year set and asserts whether the calculated expected responses match the actual responses returned. This can be used for any given regime and financial year so long as the rules set exists.

Both the Postman collections and the Batch files point to specific Rules Service financial year sets. The correct combination of Batch File and Postman collection must be used or the tests will fail. In some cases the Rules set won't have changed from one financial year to the next and so there is no need to change the Batch file calculation data and we can use the same file to individually test against multiple financial years. E.g If I wanted to test the Waste rules set for Financial Year 2018-2019, I would select the Postman collection for 18-19 and the pre 19-20 Batch file. If I wanted to test the following financial year, I would choose the post 19-20 Batch file. 

### Batch File Structure
Columns B to O in the `Inputs` tab are the data item inputs in each iterative request to the Rules set. This test covers both business logic scenarios and multiple permutations of inputs to the Rules service. 

Columns P to Z define the expected responses from the Rules service, with the charge value itself shown in column P. The expected responses are calculated for all charge types in the `Calcs` tab. The `Calcs` tab references the `Chg_Factors` (or `BaselineChargeLookup` for all other regimes outside Water Resources) and `Reduction_Values` tabs in deriving the expected responses. These factors and reduction values must of course be in-line with those used in the rules service for the financial year being tested.

### Future Financial years

In order to test future financial year rules sets in the projects current form, you would have to create a new Postman collection which includes the updated URL for the Rules set being tested. The tests would stay the same and can be copied from previous collections. And depending on whether the Rules set has changed for the new financial year, potentially create a new Batch file. 

> If the Rules set _**has not**_ changed then the latest Batch file can be used. 

> If the Rules set _**has**_ changed for a new financial year you would have to update the master Batch work sheet `Chg_Factors` & `Reduction_Values` values (or `BaselineChargeLookup` for all other regimes outside Water Resources) to include the latest values. These would then produce new values calculated and save as new Batch file (csv) ready for test. 

## Execution

To run the tests you will need to call a collection along with the corresponding batch csv file and an environment in CLI. 

For example to run the Installations 20-21 Financial Year Ruleset batch test in the non-prod environment you would need to call the following: 

```bash
newman run installations/Installations_20-21.json -d installations/Batch_Post_19-20.csv -e environments/nonprod.json
```

<img src="docs/cli.png" width="800" alt="Screenshot of test runner" />

See [commands.json](/commands.json) for a list of commands for each regime. This is particularly handy to run multiple or all rules sets consecutively. 

## How to contribute to this project

If you have an idea you'd like to contribute please log an issue.

All contributions should be submitted via a pull request.