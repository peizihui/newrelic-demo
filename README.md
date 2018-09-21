# newrelic-demo

# newrelic 安装配置
## 1. 注册

 
* 注册账号
    https://newrelic.com
* 安装对应语言的AGENT;
https://docs.newrelic.com/docs/agents/manage-apm-agents/installation/install-agent

* 安装 Infrastructure


    *  1. 查看系统配置需求 https://docs.newrelic.com/docs/infrastructure/new-relic-infrastructure/getting-started/compatibility-requirements-new-relic-infrastructure
    
    *  2.创建配置文件，并添加Infrastructure liensce_key(首次安装点击INFRASTRUCTURE TAB获取),
    echo "license_key: YOUR_LICENSE_KEY" | sudo tee -a /etc/newrelic-infra.yml
    
    *  3. 决定版本号  (Determine the distribution version number)
    
        cat /etc/os-release
        
    *   4. 创建ageng yum repo (Create the agent's yum repo)
    
    sudo curl -o /etc/yum.repos.d/newrelic-infra.repo https://download.newrelic.com/infrastructure_agent/linux/yum/el/7/x86_64/newrelic-infra.repo
    
    *   5. 更新yum缓存 （Update your yum cache）
    
    sudo yum -q makecache -y --disablerepo='*' --enablerepo='newrelic-infra'
    
    *   6. 执行安装脚本 （Run the install script）:
    
    sudo yum install newrelic-infra -y    
     
     

Review the agent requirements and supported operating systems for 64-bit architectures

## 2. 项目集成
  * Dependency  
     
        <dependency>
            <groupId>com.newrelic.agent.java</groupId>
            <artifactId>newrelic-agent</artifactId>
            <version>4.5.0</version>
            <scope>provided</scope>
        </dependency>



  * Dependency plugin
  

MAVEN内置变量
${project.build.outputDirectory} 构建过程输出目录，缺省为target/classes
        
        <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-dependency-plugin</artifactId>
                <version>2.10</version>
                <executions>
                    <execution>
                        <phase>prepare-package</phase>
                        <goals>
                            <goal>unpack-dependencies</goal>
                        </goals>
                        <configuration>
                            <includeArtifactIds>newrelic-agent</includeArtifactIds>
                            <outputDirectory>${project.build.outputDirectory}</outputDirectory>
                        </configuration>
                    </execution>
                </executions>
            </plugin>  
    
  * Maven jar plugin
    
    java agent的运行，需要定义好Premain-Class 和权限，premain-calss 是agent中指定premain（）方法的类，premain方法类似main()方法；同时也需要指定Can-Redefine-Class 和Can-Retransform-Classs 属性。  

            
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-jar-plugin</artifactId>
                <version>2.5</version>
                <configuration>
                    <archive>
                        <manifestEntries>
                            <Premain-Class>com.newrelic.bootstrap.BootstrapAgent</Premain-Class>
                            <Can-Redefine-Classes>true</Can-Redefine-Classes>
                            <Can-Retransform-Classes>true</Can-Retransform-Classes>
                        </manifestEntries>
                    </archive>
                </configuration>
            </plugin> 
  
  * Multiple main() methods
    
    目前工程中有new relic agent 的代码，该代码也有main()方法，必须指定正确的一个main方法，在
spring-boot-maven-plugin中，否则构建失败。 zhaoxi-admin 目前已经指定，不需更改。
    
            
              <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <mainClass>${start-class}</mainClass>
                    <layout>ZIP</layout>
                </configuration>
                <executions>
                    <execution>
                        <goals>
                            <goal>repackage</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>





## 3. newrelic 服务器启动
* newrelic.yml 配置

    从1. 获取license key 配置到newrelic.yml中license_key,app_name;
    
* newrelic 安装根路径下执行如下命令，启动newrelic和应用服务；
  
    
           java -javaagent:newrelic.jar -jar /root/admin.jar


    

