
# Jenkins - Overview



## Pipeline Structure
### the `pipeline` block
It is one to one mapping. One pipeline job has one jenkinsfile with one `pipeline` block.

At the `pipeline` level it is possible to specify:
- Agent: Where the job should be ran
- Environment variables: They can be declared for data, which get to be shared throughout the whole job.

#### The `stage` block
Inside the `pipeline` blocks, `stage` blocks can be defined.

At the `stage` level it is possible to specify:
- Agent: Where the job should be ran
- Environment variables: They can be declared for data, which get to be shared throughout the whole stage.

##### The  `step` block
Inside the `stage` blocks, `step` blocks are defined. Inside the step is where the actual work happens. 

```groovy
pipeline {
    agent any
    environment {
        RELEASE='20.04'
    }
    stages {
        stage('Build') {
            agent any
            environment {
                LOG_LEVEL='INFO'
            }
            steps {
                echo "Building release ${RELEASE} with log level ${LOG_LEVEL}..."
            }
        }
        stage('Test') {
            steps {
                echo "Testing. I can see release ${RELEASE}, but not log level ${LOG_LEVEL}"
            }
        }
    }
}
```


-TODO [PRINT]
-TODO [PRINT 1]


## Plugins
### workflow-aggregator plugin
- TODO PRINT 5	

## Acessing/Using Enviroment Variable in Strings
- TODO [PRINT 2]

## The `when` block
The `when` directive allows the Pipeline to determine whether the stage should be executed depending on the given condition.

### Built-in Conditions
- branch
- buildingTag
- changelog
- changeset
- changeRequest
- environment
- equals
- expression
- tag
- not
- allOf
- anyOf
- triggeredBy

#### `branch`
Execute the stage when the branch being built matches the branch pattern. Ex: `when { branch 'master' }`.
The optional parameter `comparator` may be added after an attribute to specify how any patterns are evaluated for a match: `when { branch pattern: "release-\\d+", comparator: "REGEXP"}`
- "REGEXP"
- "EQUALS"
- "GLOB" (default) 

#### `expression`
Execute the stage when the specified Groovy expression evaluates to true, for example:  `when { expression { return params.DEBUG_BUILD } }`

#### `changeRequest`
Executes the stage if the current build is for a "change request" (a.k.a. Pull Request on GitHub and Bitbucket, Merge Request on GitLab, Change in Gerrit, etc.).

#### `allOf`
Execute the stage when all of the nested conditions are true.

### Evaluating when before the input directive
By default, the when condition for a stage will not be evaluated before the input, if one is defined. However, this can be changed by specifying the `beforeInput` option within the when block. If `beforeInput` is set to true, the when condition will be evaluated first, and the input will only be entered if the when condition evaluates to true.

```groovy
pipeline {
    agent none
    stages {
        stage('Example Deploy') {
            when {
                beforeInput true
                branch 'production'
            }
            input {
                message "Deploy to production?"
                id "simple-input"
            }
            steps {
                echo 'Deploying'
            }
        }
    }
}
```

### Evaluating when before the options directive
By default, the `when` condition for a `stage` will be evaluated after entering the `options` for that `stage`, if any are defined. However, this can be changed by specifying the `beforeOptions` option within the `when` block. If `beforeOptions` is set to `true`, the `when` condition will be evaluated first, and the `options` will only be entered if the `when` condition evaluates to true.

```groovy
pipeline {
    agent none
    stages {
        stage('Example Deploy') {
            when {
                beforeOptions true
                branch 'testing'
            }
            options {
                lock label: 'testing-deploy-envs', quantity: 1, variable: 'deployEnv'
            }
            steps {
                echo "Deploying to ${deployEnv}"
            }
        }
    }
}
```

### Multiple condition and nested condition - Pipeline Example
```groovy
pipeline {
    agent any
    stages {
        stage('Example Build') {
            steps {
                echo 'Hello World'
            }
        }
        stage('Example Deploy') {
            when {
                branch 'production'
                anyOf {
                    environment name: 'DEPLOY_TO', value: 'production'
                    environment name: 'DEPLOY_TO', value: 'staging'
                }
            }
            steps {
                echo 'Deploying'
            }
        }
    }
}
```

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
This block allow us to run `stages` in a parallel way.
```groovy
stage('Parallel Stage') {
            when {
                branch 'master'
            }
            failFast true
            parallel {
                stage('Branch A') {
                    agent {
                        label "for-branch-a"
                    }
                    steps {
                        echo "On Branch A"
                    }
                }
                stage('Branch B') {
                    agent {
                        label "for-branch-b"
                    }
                    steps {
                        echo "On Branch B"
                    }
                }
	    }
}
```

## The `post` blocks

The **post** section defines one or more additional steps that are run upon the completion of a Pipeline’s or stage’s run (depending on the location of the post section within the Pipeline). post can support any of the following post-condition blocks: `always, changed, fixed, regression, aborted, failure, success, unstable, unsuccessful, and cleanup`. 

- always - Run the steps in the post section regardless of the completion status of the Pipeline’s or stage’s run.

```groovy
post {
	success {
		archiveArtifacts 'somefile.txt'
		// this will take a file from the workspace and save it as an archive inside my jenkins job
	}
}
```

### `archiveArtifacts`

`archiveArtifacts 'somefile.txt'` -  this will take a file from the workspace and save it as an archive inside my jenkins job

## The `environmet` block
The `environment` directive specifies a sequence of key-value pairs which will be defined as environment variables for all steps, or stage-specific steps, depending on where the `environment` directive is located within the Pipeline.

```groovy
stage('Build') {
     
     environment {
        VERSION_SUFFIX = getVersionSuffix()
     }
     steps {
        echo "Building version: ${VERSION} with suffix: ${VERSION_SUFFIX}"
             
    }
}
```
## The `dir` block
Change current directory. Any step inside the `dir` block will use this directory as current and any relative path will use it as base path.

Use WORKSPACE environment variable to change workspace directory.

```groovy
dir("${env.WORKSPACE}/aQA"){
    sh "pwd"
}
```

```groovy
 stage('Unit Test') {
            steps {
              dir('./m3/src') {
                sh '''
                    dotnet test --logger "trx;LogFileName=Pi.Math.trx" Pi.Math.Tests/Pi.Math.Tests.csproj
                    dotnet test --logger "trx;LogFileName=Pi.Runtime.trx" Pi.Runtime.Tests/Pi.Runtime.Tests.csproj
                '''
                mstest testResultsFile:"**/*.trx", keepLongStdio: true
              }
            }
        }
```

## The `writeFile` Command
```groovy
writeFile: 'test-results.txt', text: 'here goes the content'
```

## The `withCredentials` block
Allows various kinds of credentials (secrets) to be used in idiosyncratic ways. Each binding (withCredentials) will define an environment variable active within the scope of the step.

```groovy
steps {
	echo "Building release ${RELEASE} with log level ${LOG_LEVEL}..."
 	sh 'chmod +x m2/demo3/build.sh'
		// In this example 'an-api-key' was previusly defined in Jenkins Credentials
        withCredentials([string(credentialsId: 'an-api-key', variable: 'API_KEY')]) {
            sh '''
                ./m2/demo3/build.sh
               '''
       }
}
```


## The `script` block
Here groovy code can be written! 

**PS:**  Before run, every usage of the Java API in the script must be approved.

- TODO PRINT 3
- TODO PRINT 4

```groovy
...
stage("Test") {
	steps {
		script {
			if(Math.random() > 0.5) {
				throw new Exception()
			}
		}
	}
}
...
```
