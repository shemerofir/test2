
pipeline{

    
agent none
    stages{

        stage('inputUser'){

            steps{
                script{    
                     username = input (
                     parameters: [string(defaultValue: 'test', description: 'Enter User:', name: 'USER')] 
                )}

            }
        }

        stage('inputRepo'){

            steps{
                script {
                    properties([
                          
                        //Creating the parameters, make sure you have Active Choice plugin installed
                        parameters([

                            [$class: 'ChoiceParameter', 
                                //Single combo-box item select type of choice
                                choiceType: 'PT_SINGLE_SELECT', 
                                description: 'Select the Repository from the Dropdown List', 
                                filterLength: 1, 
                                filterable: false, 
                                //Important for identify it in the cascade choice parameter and the params. values
                                name: 'REPO', 
                                script: [
                                    $class: 'GroovyScript', 
                                    //Error script
                                    fallbackScript: [
                                        classpath: [], 
                                        sandbox: false, 
                                        script: 
                                            "return ['cant get repos']"
                                    ], 
                                    script: [
                                        classpath: [], 
                                        sandbox: false, 
                                        //Calling local variable with the script as a string
                                        script: 
                                        """import groovy.json.JsonSlurper
                                        def get = new URL("https://api.github.com/users/$username/repos").openConnection();
                                        def getRC = get.getResponseCode();

                                        if (getRC.equals(200)) {
                                            def json = get.inputStream.withCloseable { inStream ->
                                                new JsonSlurper().parse( inStream as InputStream )
                                            }

                                            def item = json;
                                            def names = [];

                                            item.each { repo ->
                                                names.push(repo.name);
                                            }   
                                            return names;
                                            }"""
                                        
                                    ]
                                ]
                            ]
                        ])
                    ])
                }           
            }
        }

        stage('inputBranch'){

            steps{
                script{
                    properties([

                         parameters([
                            //Cascade choice, means you can reference other choice values, like in this case, the REPO
                            //Also, re-runs this scripts every time the referenced choice value changes.
                            [$class: 'CascadeChoiceParameter', 
                                choiceType: 'PT_SINGLE_SELECT', 
                                description: 'Select the Branch from the Dropdown List',
                                name: 'BRANCH', 
                                //Referencing the repo
                                referencedParameters: 'REPO', 
                                script: 
                                    [$class: 'GroovyScript', 
                                    fallbackScript: [
                                            classpath: [], 
                                            sandbox: false, 
                                            script: "return ['null']"
                                            ], 
                                    script: [
                                            classpath: [], 
                                            sandbox: false, 
                                            //branchScript variable
                                            script: 

                                                    //Script for the branch, you can reference the previous script value witn the "REPO" variable
                                                    def branchScript = """import groovy.json.JsonSlurper
                                                    def getBranches = new URL("https://api.github.com/repos/$username/" + REPO + "/branches").openConnection();
                                                    def getRCBranches = getBranches.getResponseCode();

                                                    if (getRCBranches.equals(200)) {
                                                    def jsonBr = getBranches.inputStream.withCloseable { inStream ->
                                                            new JsonSlurper().parse( inStream as InputStream )
                                                    }

                                                        def itemBr = jsonBr;
                                                        def namesBr = [];

                                                        itemBr.each { branch ->
                                                            namesBr.push(branch.name);
                                                        } 
                                                        return namesBr;
                                                    }"""
                                    ] 
                                ]
                            ]
                        ])    
                    ])        
                }           
            }
        }





        stage('checkout scm') {
            agent any
            when{
               expression { Username!= 'null' && RepoName!='null' && BranchName!='null' }
            }
            steps {
                //Mkdir if not exist for the params.REPO
                sh "mkdir -p ${params.REPO}"
                //Changing workdir to the previous dir created
                dir(path: "${params.REPO}"){
                    //Pulling the git repo
                    git branch: "${params.BRANCH}", 
                        poll: false, 
                        url: "https://github.com/${params.USER}/${params.REPO}.git"
                }
            }
        }

    }
}
