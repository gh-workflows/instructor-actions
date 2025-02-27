## Helpful Snippets

Contexts:

    ${{ github.event_name }} - 
    This expression uses the github context to get the name of the event that triggered the workflow.

Functions:
    
    ${{ contains(github.event.head_commit.message, 'test') }} -
    This expression uses the contains() function to check if the commit message of the push event contains the word 'test'.

Operators:

    ${{ github.event_name == 'push' }} -
    This expression uses the == operator to check if the event that triggered the workflow is a push event.

Matrix strategy:

    ${{ matrix.node-version }} - 
    This expression uses the matrix context to get the current Node.js version in a matrix strategy.


## Examples of expressions and contexts in GitHub Actions.

Example of the `success()` function in github actions.

    jobs:
        build:
            runs-on: ubuntu-latest

            steps:
            - name: Checkout code
              uses: actions/checkout@v2

            - name: Run tests
              run: make test

            - name: Notify only on success
              if: success()
              run: echo "Tests passed successfully!"


Example of the `contains()` function in github actions.

    jobs:
        build:
            runs-on: ubuntu-latest

            steps:
            - name: Checkout code
              uses: actions/checkout@v2

            - name: Notify only if commit message contains 'test'
              run: echo "Running tests..."
              if: contains(github.event.head_commit.message, 'test')

Example of the `join()` function in github actions.

    jobs:
        build:
            runs-on: ubuntu-latest

            steps:
            - name: Checkout code
              uses: actions/checkout@v2

            - name: Join strings
              run: echo "${{ join(['Hello', 'GitHub', 'Actions'], ' ') }}"

In this context example, the `github.event_name` expression accesses the `event_name` property of the github context, which contains the name of the event that triggered the workflow. The echo command then prints a message that includes this event name.

There are several other contexts available in GitHub Actions, including `job`, `steps`, `runner`, `strategy`, `matrix`, and `secrets`. Each of these contexts provides access to a different set of data related to the workflow run.

    jobs:
        build:
            runs-on: ubuntu-latest

            steps:
            - name: Checkout code
              uses: actions/checkout@v2

            - name: Print event name
              run: echo "This workflow was triggered by a ${{ github.event_name }} event."

Example of a `matrix` in github actions. The strategy block defines a `matrix strategy`. A `matrix strategy` allows you to run the job multiple times with different configurations. In this case, the `matrix strategy` is used to run the job on three different versions of Node.js: 12.x, 14.x, and 16.x. The job will run once for each version specified in the matrix.

    jobs:
        build:
            runs-on: ubuntu-latest

            strategy:
            matrix:
                node-version: [12.x, 14.x, 16.x]

            steps:
            - name: Checkout code
              uses: actions/checkout@v2

            - name: Use Node.js ${{ matrix.node-version }}
              uses: actions/setup-node@v2
              with:
                node-version: ${{ matrix.node-version }}

            - name: Install dependencies
              run: npm ci

            - name: Run tests
              run: npm test
