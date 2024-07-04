GitHub Actions is a Continuous Intgeration and continuous deployment (CI/CD) pipeline that helps you automate your app deployment. GitHub Actions use workflow, which is triggered by an **event** that starts the workflow process. Each workflow has a **runner** environment that executes some sort of purpose called **jobs**; the runner can contain one more job. The job themselves can also have smaller tasks: _steps_ which are scripts you defined, or _actions_, which are reusable extensions to simplify your workflow.

- Workflow: an automated process that completes one or more predefined tasks, which is triggered when an event occurs. These events are either by an action caused in repo, or scheduled events.
- Events: specific activity that occurs in a repo, such as pull request, issue, or push commits.
- Jobs: a set of _steps_ or _actions_ (see the definition of italics in intro) that is completed within one container, or _runner_.
- Actions: custom application script that does complex, but repetitive tasks to make pipeline easier to maintain and deploy.
- Runners: environment that runs your workflow when triggered.

### Understanding GitHub Actions concepts and example

Each worflow file is an YAML file that is stored under `.github/workflows/`.

Each jobs can share information and data completed from one to another using artifacts. Here's an example of uploading data from one job and downloading it to another job:

```yaml
jobs:
	example-job:
		name: Save Output
		runs-on: ubuntu-latest
		steps:
			- shell: bash
			  run: |
			  	expr 1 + 1 > output.log
			- name: Upload output file
			  uses: actions/upload-artifact@v4
			  with:
			  	name: output-log-file
				path: output.log
```

```yaml
jobs:
	example-job:
		runs-on: ubuntu-latest
		steps:
			- name: download a single artifact
			  uses: actions/download-artifact@v4
			  with:
			  	name: output-log-file
```
