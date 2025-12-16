# CDA Testing

## Purpose
The purpose of this SOP is to provide guidance on how to test CDA deployed in CloudOne.

## Prequirements
- Clone of the [cda-test](https://github.com/CancerDataAggregator/cda-test.git) repo on your local machine
    - > **_NOTE:_** Follow the "Prerequisites" section of the repo's README.md

## Procedures
1. In a terminal window: 
    1. Navigate to the cda-test folder that you cloned from the repository
    1. Install CDA Python
        ``` bash
        python3 cda_manage.py --install cdapython
        ```
        - > **_NOTE:_** This will also update cdapython so run this even if you've previously installed it
    1. Launch Jupyter Lab
        ``` bash
        python3 cda_manage.py --launch jupyter
        ```
        - > **_NOTE:_** Keep the terminal running while using Jupyter
1. In a browser:
    1. navigate to https://127.0.0.1:8888 to pull up the Jupyter Lab page
    1. Open the notebooks folder
    1. Select the notebook you'd like to use for testing (or create a new one)
    1. Ensure cdapython is pointed to the API you'd like to test against via:
        ``` python
        from cdapython import set_api_url
        set_api_url('https://INSERT_API_URL_HERE')
        ```
        - > **_NOTE:_** Some notebooks still use the old ```os.environ['CDA_API_URL'] = "https://OLD_API_URL_METHOD"``` method to set this up but use the new method listed above instead
    1. Run the cells in the notebook and ensure everything is working as expected

## FAQs
The **cda-test** repo is more capable than what is described in this SOP, please refer to the README.md file in the repo for guidance on further use.