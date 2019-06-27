node {
    /* 定义构建参数 */
    properties ([parameters([string(defaultValue: '', description: '请输入要发布的代码标签', name: 'GIT_TAG'),string(defaultValue: '', description: '请输入要发布的版本', name:'VERSION'), string(defaultValue: '', description: '请输入更新内容', name:'RELEASE_NOTE')])])
    if (params.GIT_TAG == '' || params.VERSION == '') {
        error '参数输入错误'
    }

    stage('Build') {
        checkout([$class: 'GitSCM', branches: [[name: "refs/tags/${params.GIT_TAG}"]], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx', url: 'git@xxx.xxx.xxx.xxx:xxxx/example.git']]])

        dir ('example') {
            /* 构建镜像 */
            def customImage = docker.build("example-group/example:${params.VERSION}")

            /* hub.xxxx.cn是你的Docker Registry */
            docker.withRegistry('https://hub.xxxx.cn/', 'docker-registry') {
                /* Push the container to the custom Registry */
                customImage.push()
                customImage.push('latest')
            }
        }
    }

    stage('Deploy-Testing-Env') {
        if (currentBuild.result == null || currentBuild.result == 'SUCCESS') {
            /* 如果Jenkins服务器和部署的服务器不是同一台，可以使用ansible或者其他远程的方式调用部署 */
            /* 强制删除原来的容器 */
            sh 'ansible ciserver -a "docker rm -f example"'
            /* 从Registry拉取镜像 */
            sh 'ansible ciserver -a "docker pull hub.xxxx.cn/example-group/example:latest"'
            /* 运行容器 */
            sh 'ansible ciserver -a "docker run -d --name example hub.xxxx.cn/example-group/example:latest"'

            /* 发送邮件 */
            mail bcc: '', body: "XXXX已部署\n地址：http://xxx.xxx.xxx/xxx\n\n代码标签：${params.GIT_TAG}\n更新内容：\n${params.RELEASE_NOTE}\n\n----------\n本邮件为自动发送\nBy Jenkins", cc: '', from: 'jenkins@xxx.xxx', replyTo: '', subject: "【Jenkins】测试环境已更新${params.VERSION}", to: 'xxxx@xxx.xxx'
        }
    }

    stage('Test') {
        sh 'echo "Run automated test"'
    }
}
