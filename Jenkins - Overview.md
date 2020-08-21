
# Jenkins - Overview

## The `input`  block
When the build hits an input block, it will wait for some sort of user confirmation.

```groovy
stage("Deploy"){

	input {
		message 'Deploy?'
		// Button that will appears when this step is reached
		ok 'Do it'
		parameters {
			string(name: 'TARGET_ENVIRONMENT', defaultValue: 'PROD', description: 'Target deployment environment')
		}
	}
}
```
### Setting Timeout for `input` block

```groovy
timeout(time:2, unit:"HOURS"){
	input(message: "Go ahead with this deploy?")
}
```

## The `parallel` block


## The `writeFile` Command
