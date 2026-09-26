// ============================================================
// jzo2o 项目 CI 流水线（第一次版）
// 流程：拉取代码 → 构建 framework → 安装 API → 打包业务服务 → 归档 jar
// 讲解：Jenkinsfile 就是"用代码描述的构建流程"，放进仓库后，
//       任何人任何机器拿到代码都能一键复现同样的构建——这就是 CI 的核心
// ============================================================
pipeline {
    // 在 Jenkins 控制器本机上执行（学习阶段够用，以后可扩展专属构建节点）
    agent any

    options {
        timestamps()               // 日志每行加时间戳，方便算每阶段耗时
        disableConcurrentBuilds()  // 同一任务排队执行，避免并发写坏共享仓库
    }

    environment {
        // 关键一步：切换到 JDK 11 编译项目
        // （容器默认 JAVA_HOME 指向 21，那是给 Jenkins 本体用的）
        JAVA_HOME = '/opt/jdk11'
        PATH = "${JAVA_HOME}/bin:${env.PATH}"
    }

    stages {
        stage('01 拉取代码') {
            steps {
                // 按任务配置里填的仓库地址和分支检出代码
                checkout scm
            }
        }

        stage('02 构建基础框架') {
            steps {
                // 用 install 而不是 package：把 framework 的 SNAPSHOT 构件
                // 装进本地 Maven 仓库，业务模块才能引用到
                // （本地仓库挂载自宿主机，你本机 IDE 也能直接复用）
                sh 'cd jzo2o-framework && mvn -B clean install -DskipTests'
            }
        }

        stage('03 安装API模块') {
            steps {
                sh 'cd jzo2o-code/jzo2o-api && mvn -B clean install -DskipTests'
            }
        }

        stage('04 打包业务服务') {
            steps {
                // 跳过测试的原因：现有 8 个测试全是 @SpringBootTest，
                // 依赖 Nacos/MySQL 等中间件环境，CI 里跑必失败；
                // 进阶阶段用 Testcontainers 解决，届时去掉 -DskipTests
                sh 'cd jzo2o-code/jzo2o-foundations && mvn -B clean package -DskipTests'
            }
        }

        stage('05 归档构建产物') {
            steps {
                // 把 jar 存进 Jenkins，构建历史页可直接下载
                archiveArtifacts artifacts: 'jzo2o-code/jzo2o-foundations/target/*.jar',
                                 fingerprint: true
            }
        }
    }

    post {
        success { echo '✅ CI 构建成功：代码可编译、可打包' }
        failure { echo '❌ CI 构建失败：点开 Stage View 和 Console Output 定位' }
        always  {
            // 收集测试报告（当前跳过测试为空，为以后接入测试预留）
            junit allowEmptyResults: true,
                  testResults: '**/target/surefire-reports/*.xml'
        }
    }
}
