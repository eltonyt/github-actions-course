# GitHub Actions: The Complete Guide from Beginner to Expert - Notes
### Introduction

- Github Repo - ‣
- Example E2E Repo - ‣

### Tools and Initial Setup

- Node.js
    - v24.11.0
- nvm(Optional - Not Used for my project) - Manage the version of Node.js
- asdf(Optional - Not Used for my project) - Manage the version of every binary
- YML - For Github Workflow
    - Key-Value Pairs
    - Value: String, Number, Boolean, or Array
        - Array Example - with leading `-`
            - Value can be primitive value
            - Value can also be dictionary(object)
            
            ```java
            color:
              - name:red
                hex: 00ff00
              - name: blue
            	  hex: 22aa00
            ```
            
    - Can be in multiple level (Value is another dictionary in this case)
    - 2 Spaces as indent

### Github Actions Building Blocks

- Workflow, Jobs & Steps
    - Workflows
        - are defined at the repository level
        - Define which triggers actually start the workflow
        - **Are composed of one or more jobs**
    - Jobs
        - are defined at the workflow level
        - Define in which execution environment they are run
        - **are composed of one or more steps**
        - **Run in parallel by default**
    - Steps
        - Are defined at the job level
        - Define the actual script or **Github Action** that will be executed
        - **Run sequentially by default**
        - Contains an array of scripts that will run
    - Example
        - Jobs run with independent virtual matchines
        
        ![image.png](attachment:6d5b3156-ba7a-4d8d-ac57-04e104f1393c:image.png)
        
- Practice
    
    ```kotlin
    name: 01 - Building Blocks
    on: push
    jobs:
      echo-hello:
        runs-on: ubuntu-latest
        steps:
          - run: echo "Hello, World!"
      echo-goodbye:
        runs-on: ubuntu-latest
        steps:
          - name: Successful step
            run: |
              echo "I will fail"
              exit 0
          - name: Say goodbye
            run: echo "Goodbye!"
    
    ```
    

### Workflow Events

- **Triggering workflows in multiple ways**
- There are many ways we can trigger Github workflows:
    - Reference - https://docs.github.com/en/actions/reference/workflows-and-actions/events-that-trigger-workflows
    - **Repository Event**
        - `push`
            - Triggered when someone pushes to the repo
        - `issues`
            - Triggered by a variety of events related to issues
        - `pull_request`
            - Triggered by a variety of events related to PRs
        - `pull_request_review`
            - Triggered by a variety of events related to PR reviews (submitting, editing, deleting, etc)
        - `fork`
            - Triggered when your repository is forked
        - `...`
    - **Manual Trigger**
        - Triggered via the UI
            - Actions tab in Github
        - Triggered via an API call
        - Triggered from another workflow
    - **Schedule**
        - Run as a cron job
- You can access to the github action workflow context using `${{ xxxxx }}`
    - Example: `${{ github.event_name}}`
- If you want to have the workflow run on multiple events
    - `workflow_dispatch` enables the trigger from Github UI
    
    ```java
    on:
      push:
      pull_request:
      schedule:
        - cron: "*/5 * * * *" # Every 15 minutes
      workflow_dispatch:
    ```
    

### Workflow Runners

- Concept: Virtual Servers that execute jobs from workflows
- Github-hosted (standard)
    - Windows & Ubuntu (2 Cores, 7GB RAM, 14GB Storage)
    - MAC (3 cores, 14GB RAM, 14GB Storage)
    - Managed Service
    - A VM is scoped to a job: **steps share the VM, but jobs don’t (by default, each job receives a clean VM instance)**
- Self-hosted (DO NOT use in public repositories)
    - Run workflows on (almost) any infrastructure of your choice
    - Full control over the VM infrastructure
    - It’s not managed, meaning we need to take care of OS patching, software updates, among other ops tasks
    - Can be added at the repository, organization, or enterprise level
    - Jobs do not necessarily have to run on clean instances
- 实操
    - **runs-on 这个位置定义 ubuntu/windows/macOS**

### Actions (Scripts)

- Interesting - Looks like the dependencies that will be used
- Why use actions
    - Prevents code duplication and reduces the chances of mistakes
    - Can be configured via the **with** key-value pair
        - Enables great flexibility and reusability
    - Can be combined with other steps
        - Can be used to encapsulate setup tasks before running other commands
    - We can create our own actions for public or private use
        - We are not restricted only to the actions available at the **Marketplace (github.com/marketplace)**
- Example
    
    ```yaml
    name: 04 - Using Actions
    on: push
    jobs:
      build:
        runs-on: ubuntu-latest
        steps:
          - name: Checkout Code
            uses: actions/checkout@v4
          - name: Printing Folders
            run: ls
          - name: Setup Node
            uses: actions/setup-node@v4
            with:
              node-version: "24.11.0"
          - name: Install Dependencies
            run: npm ci
            working-directory: 04-using-actions/react-app
          - name: Run Tests
            run: npm run test
            working-directory: 04-using-actions/react-app
    
    ```
    

### Event Filters

- Specify **under which conditions** a specific event triggers our workflow
- Most triggers can be configured to specify when they should run (And Condition)
    - `branches` → Specifies which branch must match in order for the workflow to execute
    - `branches_ignore`
    - `tags`
    - `tags_ignore`
    - `paths`
    - `paths_ignore`
    - `etc`
- Reference → Github Actions → Workflow Syntax
- Example
    
    ```yaml
    name: 05 - 1 - Event Filters and Activity Types
    on:
      pull_request:
        types: [opened, synchronize]
        branches:
          - main
    jobs:
      echo:
        runs-on: ubuntu-latest
        steps:
          - run: echo "This job runs only for opened and synchronize pull request events on the main branch."
    
    ```
    

### Activity Types

- Specify which types of certain triggers execute our workflow
- Many triggers have multiple activity types we can leverage
    - `open` - Runs the workflow whenever a PR is opened
    - `synchronize` - Runs the workflow whenver a new commit is pushed to the HEAD ref of the PR
    - `closed`
    - `assigned`
    - `labeled`
    - `edited`
    - etc.
- Reference → Github Actions → Events that trigger workflows
- Example
    
    ```yaml
    name: 05 - 2 - Event Filters and Activity Types
    on:
      pull_request:
        types: [closed]
        branches:
          - main
    jobs:
      echo:
        runs-on: ubuntu-latest
        steps:
          - run: echo "Running whenever a PR is closed"
    
    ```
    

### Workflow Contexts

- Access Information about runs, variables, jobs, and much more
- Github provides multiple sources of data in different contexts so that we can easily provide all the necessary information to our workloads
    - github
        - Commit SHA
        - Event name
        - Ref of branch or tag triggering the workflow
    - env
        - Contains variables that have been defined in a workflow, job, or step. Changes based on which part of the workflow is executing
    - inputs
        - Contains input properties passed via the keyword `with` to an action, to a reusable workflow, or to a manually trigger workflow
    - vars
        - Contains custom configuration variabnle set at the organization, repository, and environment levels
    - secrets
    - matrix
    - needs
    - etc
- Reference - Github Actions → Contexts
- Access to the context - `${{ <context> }}`
- Example
    
    ```yaml
    name: 06 - Contexts
    run-name: 06 - Contexts | DEBUG - ${{ inputs.test }}
    env:
      MY_WORKFLOW_VAR: "workflow"
      MY_OVERWRITTEN_VAR: "workflow"
    on:
      #   push:
      workflow_dispatch:
        inputs:
          test:
            description: "Enable debug mode"
            type: boolean
            default: false
    jobs:
      echo-data:
        runs-on: ubuntu-latest
        steps:
          - name: Display Information
            run: |
              echo "Event Name: ${{ github.event_name }}"
              echo "Ref: ${{ github.ref }}"
              echo "SHA: ${{ github.sha }}"
              echo "Repository: ${{ github.repository }}"
              echo "Actor: ${{ github.actor }}"
              echo "Workflow: ${{ github.workflow }}"
              echo "Run ID: ${{ github.run_id }}"
              echo "Run Number: ${{ github.run_number }}"
          - name: Retrieve Variable
            run: |
              echo "MY_VAR is set to: ${{ vars.MY_VAR }}"
          - name: Print Env Variables
            run: |
              MY_OVERWRITTEN_VAR="step"
              echo "Workflow env: $MY_WORKFLOW_VAR"
              echo "Overwritten env: $MY_OVERWRITTEN_VAR"
          - name: Print Env Variables - Duplicated
            run: |
              echo "Workflow env: $MY_WORKFLOW_VAR"
              echo "Overwritten env: $MY_OVERWRITTEN_VAR"
    
    ```
    

### Expressions

- Can be used to reference information from multiple sources within the workflow
- Must use the `${{ <expression> }}` syntax - similar to the configuration key
- Can be any combination of
    - Literal values - Strings, numbers, booleans, null
    - Context Values - Values passed via the many workflow contexts
    - Functions - Built-in functions provided by Github Actions
- Support the use of functions and operators such as `!`, `<`, `>` , `!=`, `&&` , `||` , and many others.

### Variables

- Single Workflow
    - Important:
        - Lower Level overwrites Higher Level
        - `echo "Hello $NAME"`
    - Workflow Level
        
        ```yaml
        env:
        	NAME: xxx
        ```
        
    - Job Level
    - Step Level
- Multiple workflows
    - Important:
        - Lower Level overwrites Higher Level
        - `echo "Hello ${{ vars.NAME }}"`
    - Organization Level
    - Repository Level
    - Environment
    - Example
        
        ```yaml
        name: 08 - Using Variables
        on:
          #   push:
          workflow_dispatch:
        env:
          WORKFLOW_VAR: I am a workflow env var
          OVERWRITTEN: I will be overwritten
          UNDEFINED_VAR_WITH_DEFAULT: ${{ vars.UNDEFINED_VAR_WITH_DEFAULT || 'default value' }}
        jobs:
          echo:
            runs-on: ubuntu-latest
            env:
              JOB_VAR: I am a job env var
              OVERWRITTEN: I have been overwritten at the job level
            steps:
              - name: Echo Variables
                env:
                  STEP_VAR: I am a step env var
                  step_var2: I am another step var
                run: |
                  echo "Step env var: $STEP_VAR"
                  echo "Step env var 2: $step_var2"
                  echo "Job env var: $JOB_VAR"
                  echo "Workflow env var: $WORKFLOW_VAR"
                  echo "Overwritten: $OVERWRITTEN"
              - name: Overwrite Job Variable
                env:
                  OVERWRITTEN: I have been overwritten at the step level
                run: |
                  echo "Step env var: $OVERWRITTEN"
          echo2:
            runs-on: ubuntu-latest
            steps:
              - name: Print Variables
                run: |
                  echo "Repo var: ${{ vars.PRACTICE9_REPOSITORY_VAR}}"
          echo-prod:
            runs-on: ubuntu-latest
            environment: prod
            steps:
              - name: Print Prod Variables
                run: |
                  echo "Org overwritten var: ${{ vars.OVERWRITTEN_VAR}}"
                  echo "Repo var: ${{ vars.PRACTICE9_REPOSITORY_VAR}}"
                  echo "Environment var: ${{ vars.TARGET_VAR }}"
          echo-undefined:
            runs-on: ubuntu-latest
            steps:
              - name: Print Undefined Variables
                run: |
                  echo "Org var: $UNDEFINED_VAR_WITH_DEFAULT"
        
        ```
        

### Functions

- Out-of-the-box functions to model complex behavior
- Two Types
    - General purpose functions
        - Provides a set of utility functions to interact with data from multiple contexts and model more complex behavior, such as more advanced control of the workflow and job execution.
        - i.e. `contains(), startsWith(), endsWith(), fromJSON(), toJSON()`
    - Status check functions
        - Provides a set of functions allow using the status of the workflow, previous jobs or steps to define whether a certain job or step should be executed.
        - i.e. `success(), failure(), always(), cancelled()`
- Reference: Github Actions → Expressions
- Example
    
    ```yaml
    name: 09 - Using Functions
    on:
      #   pull_request:
      workflow_dispatch:
    jobs:
      echo1:
        runs-on: ubuntu-latest
        steps:
          - name: Print PR title
            run: |
              echo "PR title is: ${{ github.event.pull_request.title }}"
          - name: Print PR labels
            run: |
              echo "PR title is: ${{ github.event.pull_request.labels }}"
          - name: Bug step
            if: ${{ failure() }} && ${{ !cancelled() }}
            run: echo "I am a bug fix"
          - name: Sleep for 20 seconds
            run: sleep 20
          - name: Failing step
            run: exit 1
          - name: I will be skipped
            if: ${{ success() }}
            run: |
              echo "I will print if previous steps succeed."
          - name: I will execute
            if: ${{ failure() }}
            run: echo "I will execute"
          - name: I will execute
            if: ${{ !cancelled() }}
            run: echo "I will always print, except when the workflow is cancelled."
          - name: I will execute when cancelled
            if: ${{ cancelled() }}
            run: echo "I will execute when the workflow is cancelled."
    
    ```
    

### Controlling the Execution Flow

- Execute jobs and steps conditionally, and set dependencies between jobs
- Types:
    - Standard Execution
        - Downstream jobs and steps execute if and only if upstream jobs and steps succeed
            
            ```yaml
            jobs:
            	jobs1:
            		steps:
            			- id: step1
            			- id: step2
            			- id: step3
            ```
            
    - Conditional Execution (`if` key)
        - Downstream jobs and steps can be executed even if the upstream ones fail:
            
            ```yaml
            jobs:
            	jobs1:
            		steps:
            			- id: step1
            			- id: step2
            			- id: step3
            			  if: ${{ !cancelled() }}
            ```
            
    - Sequential job execution via the `needs` key:
        - Non-dependent execution
            - All jobs are executed in parallel by default
                
                ```yaml
                jobs:
                  job1:
                  job2:
                  job3:
                ```
                
        - Dependent execution
            - jobs wait until their dependencies successfully execute
                
                ```yaml
                job:
                  job1:
                  job2:
                  job3:
                    needs:
                      - job1
                      - job2
                  job4:
                    needs: job1
                ```
                
- Example
    
    ```yaml
    name: 10 - Controlling the Execution Flow
    on:
      workflow_dispatch:
        inputs:
          pass-unit-tests:
            type: boolean
            description: "Did the unit tests pass?"
            default: false
    jobs:
      lint-built:
        runs-on: ubuntu-latest
        steps:
          - name: Lint and build
            run: echo "Linting and building project"
      unit-tests:
        runs-on: ubuntu-latest
        continue-on-error: true
        steps:
          - name: Running unit tests
            run: echo "Running tests..."
          - name: Failing tests
            if: ${{ !inputs.pass-unit-tests }}
            run: exit 1
      deploy-nonprod:
        runs-on: ubuntu-latest
        needs:
          - lint-built
          - unit-tests
        steps:
          - name: Deploying to nonprod
            run: echo "Deploying to nonprod..."
      e2e-tests:
        runs-on: ubuntu-latest
        needs: deploy-nonprod
        steps:
          - name: Running E2E tests
            run: echo "Running E2E tests"
      load-tests:
        runs-on: ubuntu-latest
        needs: deploy-nonprod
        steps:
          - name: Running load tests
            run: echo "Running load tests"
      deploy-prod:
        runs-on: ubuntu-latest
        needs:
          - e2e-tests
          - load-tests
        steps:
          - name: Deploying to production
            run: echo "Deploying to production..."
    
    ```
    

### Inputs

- Provide information to customize workflows and actions
- Inputs enable us to request specific information from the workflow or action caller and use this information at runtime
- Github Action
    
    ```yaml
    inputs:
      url:
        description: "..."
        required: true
      max_trials:
        description: '...'
        required: false
        default: '60'
    ```
    
- Caller
    
    ```yaml
    jobs:
      job1:
        steps:
          - name: Ping URL
            uses: ping-url-example@v1
            with:
              url: https://www.google.com
              max-trials: 10
    ```