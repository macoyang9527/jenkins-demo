node('joker-jnlp') {
    stage('Clone') {
      echo "1.Clone Stage"
      git url: "https://github.com/macoyang9527/jenkins-demo.git"
    }
    stage('Build') {
      echo "2.Build Docker Image Stage"
     script {
        build_tag = sh(returnStdout: true, script: 'git rev-parse --short HEAD').trim()
        }
      sh "docker build -t registry.dannyedu.com/library/jenkins-demo:${build_tag} ." 
      echo "${build_tag}__________________________-"
    }
    stage('Push') {
        echo "3.Push Docker Image Stage"
        withCredentials([usernamePassword(credentialsId: 'HarborAuth', passwordVariable: 'dockerHubPassword', usernameVariable: 'dockerHubUser')]) {
        sh "docker login -u ${dockerHubUser} -p ${dockerHubPassword} registry.dannyedu.com"
        sh "docker push registry.dannyedu.com/library/jenkins-demo:${build_tag}"
    }
    }
    stage('YAML') {
    echo "4. Change YAML File Stage"
    def userInput = input(
        id: 'userInput',
        message: 'Choose a deploy environment',
        parameters: [
            [
                $class: 'ChoiceParameterDefinition',
                choices: "Dev\nQA\nProd",
                name: 'Env'
            ]
        ]
    )
    echo "This is a deploy step to ${userInput.Env}"
    sh "sed -i 's/<BUILD_TAG>/${build_tag}/' k8s.yaml"
    sh "sed -i 's/<BRANCH_NAME>/${env.BRANCH_NAME}/' k8s.yaml"
    }
    
    stage('Deploy') {
        echo "5. Deploy Stage"
        if (env.BRANCH_NAME == 'master') {
            input "确认要部署线上环境吗？"
        }
        sh "sed -i 's/<BUILD_TAG>/${build_tag}/' k8s.yaml"
        sh "sed -i 's/<BRANCH_NAME>/${env.BRANCH_NAME}/' k8s.yaml"
        sh "kubectl apply -f k8s.yaml --record"
    }
}
