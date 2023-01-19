# How it works

Every time a dev opens a new Pull Request or changes a Pull Request, gitStream is triggered. Next, the `.cm` file is used to determine which automatic actions are invoked based on their conditions. The resulting actions use Git provider API to achieve the desired outcome.

The next steps are common practice of changing code in repo:

1. Create a feature branch 
2. Commit changes  
3. Push branch to remote repo 
4. Open Pull Request 


```mermaid
gitGraph
   commit id: "A"
   commit id: "B"
   branch feature
   commit id: "C"
   commit id: "D"
   commit id: "E"
   checkout main
   commit id: "F"
   commit id: "G"
```

When a new PR is opened, several actors are running:

1. Git provider API
2. gitStream **service** which you have installed from the marketplace 
3. gitStream **CI/CD** script that is placed per the Git provider requirements 
4. gitStream **agent** that is executed by the CI/CD script

Once a new PR is opened (or changed) the following process occurs:

1. gitStream gets event for the new PR 
2. gitStream calls the **CI/CD** script
3. The installed action pulls and runs gitStream action `linear-b/gitstream-github-action@v1`. 
4. This action runs locally in the repo and relies on 
5. The current branch is used to check which automations are valid from `.cm/gitstream.cm`
6. The action calls to gitStream app with metadata
7. gitStream app returns results
8. The list of applicable automations are sent to the gitStream service
9. gitStream service iterates over the automations and invokes each action using Git provider APIs
10. The PR gets updated according to the desired automations

At the end, the PR is ready for further review or merge.

The following diagram describes the flow:

``` mermaid
sequenceDiagram
  autonumber
  Git provider API->>gitStream app: new PR
  gitStream app->>gitStream CI/CD script: run 
  activate gitStream CI/CD script
  gitStream CI/CD script->>gitStream CI/CD script: pull agent action
  gitStream CI/CD script->>gitStream in repo agent: run
  activate gitStream in repo agent
  gitStream in repo agent->>gitStream in repo agent: parse the `.cm` rules
  gitStream in repo agent->>gitStream app: API calls (metadata read-only)
  gitStream app->>gitStream in repo agent: results
  gitStream in repo agent->>gitStream app: applicable automations
  deactivate gitStream in repo agent
  deactivate gitStream CI/CD script
  loop per automation
    loop per action
      gitStream app->>gitStream app: execute action
      gitStream app->>Git provider API: update PR
    end
   end
```

## The branch diff

The branch diff is defined as the difference between the current branch and the most recent commit in the main branch, which is shared between both branches.

```mermaid
gitGraph
   commit id: "A"
   commit id: "B"
   branch feature
   commit id: "C"
   commit id: "D"
   checkout main
   commit id: "E"
   commit id: "F"
   checkout feature
   merge main
   commit id: "G"
   commit id: "H"
   checkout main
   commit id: "I"
   commit id: "J"
```

In the example above, the diff is performed between commits `H` and `F`.

## Automation results

Eventually, the gitStream app shows the following statuses:  

![Success](/assets/github_pr_check_pass.png) Success - when the applicable automation finished and PR is good to go 

![Neutral](/assets/github_pr_check_neutral.png) Neutral - when there aren't any applicable automations for the PR

![Failed](/assets/github_pr_check_fail.png) Failed - when the applicable automation finished without completion