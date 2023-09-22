# github-actions-experiments
Experiment with workflow actions.

## Pull Requests
There are examples prefixed pr- for pull requests. 

See some example GitHub event JSON structures in the files on the root named:  
` github-pr-event-{event}.json `

Nice reference to the actions:  
https://frontside.com/blog/2020-05-26-github-actions-pull_request/

The workflow triggers documentation is also helpful:  
https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows

### pr-deploy/destroy-environment
These two workflows are a pair, in a concurrency group together named 'deploy' (meaning only one job at a time can run from this pair of workflows).

#### Deploy ([pr-deploy-environment.yml](./.github/workflows/pr-deploy-environment.yml))  
The concept is to deploy a temporary environment when either of two events occurs:
1. A label 'deploy' is applied to a pull request.
2. A new commit is pushed to the base branch of a pull request having a label 'deploy'

In order to be triggered under only these conditions there are two things required: 

1. A trigger to start the workflow on one of these pull_request events:
` [labeled, synchronize] ` 
2. A condition on the label name 'deploy':
` if: contains(github.event.pull_request.labels.*.name, 'deploy') `

#### Destroy ([pr-destroy-environment.yml](./.github/workflows/pr-destroy-environment.yml))  
In order to destroy the environment, there is a matching workflow which is triggered by:
` [ closed, unlabeled ] `

And filtered using the condition:
```
    if: |
      (github.event.action == 'unlabeled' && github.event.label.name == 'deploy') 
      || 
      (github.event.action == 'closed' && contains(github.event.pull_request.labels.*.name, 'deploy'))
```
