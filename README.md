Microservices
-------------------------------------

    Pre-Reqeusites
        Spring Framework
            Spring Core, Spring Context
            Spring EL
            Spring Boot
            Spring Web (Web MVC) & REST Api
            Spring Data JPA

    Lab SetUp

        JDK 1.8
        STS 4
        Maven
        MySQL Community Server 8.0

    Introduction

        MonoLithical

            1. The entire application is in one deployment-unit.
            2. The entire application is modularized through layers.
            3. Layer are going offer loosly couple components.

            - Load Management via Scaling.
            - Interoparability
            - Management 
        
        Microservices

            1. A microservice is one isolated module of an application that is
                independetly managed as a deployment unit.
            2. A group of inter-communicating microservices form an application eco-system.

            + Load Management via Scaling.
            + Interoparability
            + Management 

            ? Decomposition
            ? Inter Communication
            ? Distributed Tracing
            ? Fault Tolerence (Fall Backs)
            ? Monitoring and Configuration

        Microservices Design Patterns

            Decomposition Design Patterns
                Decomposition by Domain
                Decomposition by Sub-domain
            Integration Design Patterns
                Api Gateway Pattern
                Aggregator Pattern
                Client Side Component Pattern
            Database Design Patterns
                Database Per Service
                Shared Database            
                Saga Pattern
                CQRS Pattern
            Observability Design Patterns
                Log Aggregator
                Performence Metrics Aggregator
                Distributed Tracing
            Cross Cutting Design Patterns
                Discovery Service 
                Circuite Breaker
                External Configuaration

        A Case Study        

            BudgetTracker

                1. Each user is represented as an AccountHolder.
                2. Each AccountHolder can save a Transaction into the system.
                3. a Transaction can be either CREDIT or DEBIT
                4. The system must be able to generate Periodic Statements.

        Micrservice Design Of BudgetTracker

            Decomposition by Domain
                suggest that each domain (module) of a monolythic ideas turns out to be a microservice.

                budgettracking 
                    accountholders service
                    txns service
                    statement service

            Decomposition by sub-domain
                how do we manage the god-classes that make existence in almost all microservices.
                a sub-domain otherwise called the context is the set of requirements of the
                current microservice. Suggested that the god-class be bounded to that context.
                Hence the term bounded-context.

                budgettracking 
                    profiles service
                        AccountHolder Entity
                            Long ahId
                            String fullName
                            String mobile
                            String mailId

                    txns service
                        AccountHolder Entity
                            Long ahId
                            Double currentBalance
                            Set<Txn> txns
                        Txn           Entity
                            Long txnId
                            String header
                            Double amount
                            TxnType type
                            LocalDate txnDate
                            AccountHolder holder

                    statement service
                        AccountHolder Model
                            Long ahId
                            String fullName
                            String mobile
                            String mailId
                            Double currentBalance

                        Txn           Model
                            Long txnId
                            String header
                            Double amount
                            TxnType type
                            LocalDate txnDate

                        Statement     Model
                            LocalDate start
                            LocalDate end
                            AccountHolder profile
                            Set<Txn> txns
                            totalCredit
                            totalDebit
                            statementBalance

            Aggregator Pattern

                a req should be processed by a microservice by collelcting 
                requried information from other microservices and compose them into one resposne.

                req foir statement ------------> statement-service ---------------> profile service
                                                                <---account holder data---
                                                                --------------------> txns service
                                                                <----list of txns-------
                                                does the composition and computation
                        <---statement obj-------  into statement obj

            Discovery Service Design Pattern

                            discovery-service
                            (spring cloud netflix eureka discovery service)
                                    ↑|
                                registration of urls 
                                and retrival of urls
                                    |↓
                        -------------------------------------
                        |               |                   |
                profile-service     txns-service     statement-service

            Api Gateway Pattern Design Pattern

                Andriod App/Angular App/ReactJS App
                    ↑↓
                api-gateway
                (spring cloud api gateway)
                    |
                    |
                    | <---->   discovery-service
                        ↑    ( netflix eureka discovery service)
                        |            ↑|
                        |        registration of urls 
                        |       and retrival of urls
                        ↓            |↓
                        -------------------------------------
                        |               |                   |                
                profile-service     txns-service     statement-service
                
            Circuite Breaker design  pattern
                    is to create a thrushold for failed requests and
                    accomadate a backing-solution / fallback machanisim to handle
                    the failed reqeust.

            Distributed Tracing

            Andriod App/Angular App/ReactJS App
                    ↑↓
                api-gateway
                (spring cloud api gateway)
                    |
                    |
                    | <---->   discovery-service
                        ↑    ( netflix eureka discovery service)
                        |            ↑|
                        |        registration of urls 
                        |       and retrival of urls
                        ↓            |↓
                        -------------------------------------
                        |               |                   |                
                profile-service     txns-service     statement-service
                    (sleuth)          (sleuth)            (sleuth)
                        |               |                      |
                        -------------------------------------------
                                    ↑↓
                        distrubuted tracing service
                                (zipkin-server)

            External Configuaration

                Andriod App/Angular App/ReactJS App
                        ↑↓
                    api-gateway
                    (spring cloud api gateway)
                        |
                        |
                        | <---->   discovery-service
                            ↑    ( netflix eureka discovery service)
                            |            ↑|
                            |        registration of urls 
                            |       and retrival of urls
                            ↓            |↓
                            -------------------------------------
                            |               |                   |                
                    profile-service     txns-service     statement-service
                        (sleuth)          (sleuth)            (sleuth)
                            |               |                      |
                            -------------------------------------------
                                        ↑↓                      ↑↓
                            distrubuted tracing service       configuaration-service 
                                    (zipkin-server)         (spring cloud config service)
                                                                    |
                                                                    |
                                                                    git-repo
                                                                        profile.properties
                                                                        txns.properties
                                                                        statement.properties
                                                                        gateway.properties

            Implementing Budget-tracker
                                                    
                Step#1  implementing decomposed services and do inter-service communication and aggregator
                    in.bta:bta-profiles
                        dependencies
                            org.springframework.boot:spring-boot-starter-web
                            org.springframework.boot:spring-boot-devtools
                            org.springframework.cloud:spring-cloud-openfeign
                            mysq1:mysql-connector-java
                            org.springframework.boot:spring-boot-starter-data-jpa
                        configuaration
                            spring.application.name=profiles
                            server.port=9100

                            spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
                            spring.datasource.username=root
                            spring.datasource.password=root
                            spring.datasource.url=jdbc:mysql://localhost:3306/bapsDB?createDatabaseIfNotExist=true
                            spring.jpa.hibernate.ddl-auto=update

                    in.bta:bta-txns
                        dependencies
                            org.springframework.boot:spring-boot-starter-web
                            org.springframework.boot:spring-boot-devtools
                            org.springframework.cloud:spring-cloud-openfeign
                            mysq1:mysql-connector-java
                            org.springframework.boot:spring-boot-starter-data-jpa
                        configuaration
                            spring.application.name=txns
                            server.port=9200

                            spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
                            spring.datasource.username=root
                            spring.datasource.password=root
                            spring.datasource.url=jdbc:mysql://localhost:3306/batxnsDB?createDatabaseIfNotExist=true
                            spring.jpa.hibernate.ddl-auto=update

                    in.bta:bta-statement
                        dependencies
                            org.springframework.boot:spring-boot-starter-web
                            org.springframework.boot:spring-boot-devtools
                            org.springframework.cloud:spring-cloud-openfeign
                        configuaration
                            spring.application.name=statement
                            server.port=9300

                Step#2  implementing discovery service and client side load balancing
                    in.bta:bta-discovery
                        dependencies
                            org.springframework.boot:spring-boot-devtools
                            org.springframework.cloud:spring-cloud-starter-netflix-eureka-server
                        configuaration
                            @EnableEurekaServer    on Application class

                            spring.application.name=discovery
                            server.port=9000

                            eureka.instance.hostname=localhost
                            eureka.client.registerWithEureka=false
                            eureka.client.fetchRegistry=false
                            eureka.client.serviceUrl.defaultZone=http://${eureka.instance.hostname}:${server.port}/eureka/
                            eureka.server.waitTimeInMsWhenSyncEmpty=0

                    in.bta:bta-profiles
                        dependencies
                            ++ org.springframework.cloud:spring-cloud-starter-netflix-eureka-client
                            ++ org.springframework.cloud:spring-cloud-starter-loadbalancer
                        configuaration
                            ++@EnableDiscoveryClient  on Application class

                            eureka.client.serviceUrl.defaultZone=http://localhost:9000/eureka/
                            eureka.client.initialInstanceInfoReplicationIntervalSeconds=5
                            eureka.client.registryFetchIntervalSeconds=5
                            eureka.instance.leaseRenewalIntervalInSeconds=5
                            eureka.instance.leaseExpirationDurationInSeconds=5

                            spring.cloud.loadbalancer.ribbon.enabled=false

                    in.bta:bta-txns
                        dependencies
                            ++ org.springframework.cloud:spring-cloud-starter-netflix-eureka-client
                            ++ org.springframework.cloud:spring-cloud-starter-loadbalancer
                        configuaration
                            ++@EnableDiscoveryClient  on Application class

                            eureka.client.serviceUrl.defaultZone=http://localhost:9000/eureka/
                            eureka.client.initialInstanceInfoReplicationIntervalSeconds=5
                            eureka.client.registryFetchIntervalSeconds=5
                            eureka.instance.leaseRenewalIntervalInSeconds=5
                            eureka.instance.leaseExpirationDurationInSeconds=5

                            spring.cloud.loadbalancer.ribbon.enabled=false

                    in.bta:bta-statement
                        dependencies
                            ++ org.springframework.cloud:spring-cloud-starter-netflix-eureka-client
                            ++ org.springframework.cloud:spring-cloud-starter-loadbalancer
                        configuaration
                            ++@EnableDiscoveryClient  on Application class

                            eureka.client.serviceUrl.defaultZone=http://localhost:9000/eureka/
                            eureka.client.initialInstanceInfoReplicationIntervalSeconds=5
                            eureka.client.registryFetchIntervalSeconds=5
                            eureka.instance.leaseRenewalIntervalInSeconds=5
                            eureka.instance.leaseExpirationDurationInSeconds=5

                            spring.cloud.loadbalancer.ribbon.enabled=false    

                Step 3: Implement API Gateway Design Pattern
                    in.bta:bta-gateway
                        dependencies
                            org.springframework.boot:spring-boot-devtools
                            org.springframework.cloud:spring-cloud-starter-api-gateway
                            org.springframework.cloud:spring-cloud-starter-netflix-eureka-client
                            org.springframework.cloud:spring-cloud-starter-loadbalancer
                        configuaration
                            @EnableDiscoveryClient          on Application class

                            spring.application.name=gateway
                            server.port=9999

                            eureka.client.serviceUrl.defaultZone=http://localhost:9000/eureka/
                            eureka.client.initialInstanceInfoReplicationIntervalSeconds=5
                            eureka.client.registryFetchIntervalSeconds=5
                            eureka.instance.leaseRenewalIntervalInSeconds=5
                            eureka.instance.leaseExpirationDurationInSeconds=5

                            spring.cloud.gateway.discovery.locator.enabled=true
                            spring.cloud.gateway.discovery.locator.lower-case-service-id=true
                            
                    in.bta:bta-discovery
                    in.bta:bta-profiles
                    in.bta:bta-txns
                    in.bta:bta-statement
                        
                Step 4: Implement Distributed Tracing Design Pattern
                    in.bta:bta-discovery
                    
                    in.bta:bta-gateway
                        dependencies
                            ++org.springframework.boot:spring-boot-starter-actuator
                            ++org.springframework.cloud:spring-cloud-starter-sleuth
                            ++org.springframework.cloud:spring-cloud-starter-zipkin : 2.2.8.RELEASE
                        
                        configuaration
                            logger.level.org.springramework.web=debug
                            management.endpoints.web.exposure.include=*
                
                    in.bta:bta-profiles
                        dependencies
                            ++org.springframework.boot:spring-boot-starter-actuator
                            ++org.springframework.cloud:spring-cloud-starter-sleuth
                            ++org.springframework.cloud:spring-cloud-starter-zipkin : 2.2.8.RELEASE
                        
                        configuaration
                            logger.level.org.springramework.web=debug
                            management.endpoints.web.exposure.include=*

                    in.bta:bta-txns
                        dependencies
                            ++org.springframework.boot:spring-boot-starter-actuator
                            ++org.springframework.cloud:spring-cloud-starter-sleuth
                            ++org.springframework.cloud:spring-cloud-starter-zipkin : 2.2.8.RELEASE
                        
                        configuaration
                            logger.level.org.springramework.web=debug
                            management.endpoints.web.exposure.include=*

                    in.bta:bta-statement
                        dependencies
                            ++org.springframework.boot:spring-boot-starter-actuator
                            ++org.springframework.cloud:spring-cloud-starter-sleuth
                            ++org.springframework.cloud:spring-cloud-starter-zipkin : 2.2.8.RELEASE
                        
                        configuaration
                            logger.level.org.springramework.web=debug
                            management.endpoints.web.exposure.include=*

                    tracing-service
                        zipkin-server
                            https://search.maven.org/remote_content?g=io.zipkin&a=zipkin-server&v=LATEST&c=exec 
                            
                            java -jar zipkin-server.jar

                Step 5: Implement Circuit Breaker Design Pattern
                    in.bta:bta-discovery  
                    in.bta:bta-gateway
                    in.bta:bta-profiles
                    in.bta:bta-txns
                        dependencies
                            ++org.springframework.cloud:spring-cloud-starter-circuitbreaker-resilience4j
                        
                        configuaration
                            resilience4j.circuitbreaker.configs.default.registerHealthIndicator=true
                            resilience4j.circuitbreaker.configs.default.ringBufferSizeInClosedState=4
                            resilience4j.circuitbreaker.configs.default.ringBufferSizeInHalfOpenState=2
                            resilience4j.circuitbreaker.configs.default.automaticTransitionFromOpenToHalfOpenEnabled=true
                            resilience4j.circuitbreaker.configs.default.waitDurationInOpenState= 20s
                            resilience4j.circuitbreaker.configs.default.failureRateThreshold= 50
                            resilience4j.circuitbreaker.configs.default.eventConsumerBufferSize= 10

                    in.bta:bta-statement
                    dependencies
                            ++org.springframework.cloud:spring-cloud-starter-circuitbreaker-resilience4j
                        
                        configuaration
                            resilience4j.circuitbreaker.configs.default.registerHealthIndicator=true
                            resilience4j.circuitbreaker.configs.default.ringBufferSizeInClosedState=4
                            resilience4j.circuitbreaker.configs.default.ringBufferSizeInHalfOpenState=2
                            resilience4j.circuitbreaker.configs.default.automaticTransitionFromOpenToHalfOpenEnabled=true
                            resilience4j.circuitbreaker.configs.default.waitDurationInOpenState= 20s
                            resilience4j.circuitbreaker.configs.default.failureRateThreshold= 50
                            resilience4j.circuitbreaker.configs.default.eventConsumerBufferSize= 10

                Step 6: External Configuaration Design Pattern
                    inTheWorkSpace> md bt-props-repo
                        //and then create these files in this directory
                            // gateway.properties
                            // profiles.properties
                            // txns.properties
                            // statement.properties
                            // move the content of 'application.properties' of each microservice into these respective files
                            
                        inTheWorkSpace> cd bt-props-repo
                        inTheWorkSpace\bt-props-repo> git init           
                        inTheWorkSpace\bt-props-repo> git add .
                        inTheWorkSpace\bt-props-repo> git commit -m "all service properties"
                    
                    in.bta:bta-discovery
                    in.bta:bta-config
                        dependencies
                            org.springframework.boot:spring-boot-devtools
                            org.springframework.cloud:spring-cloud-config-server
                            org.springframework.cloud:spring-cloud-starter-netflix-eureka-client
                        
                        configuaration  
                            @EnableDiscoveryClient
                            @EnableConfigServer             on Application class

                            spring.application.name=config
                            server.port=9090

                            spring.cloud.config.server.git.uri=file:///local/git/repo/path

                            eureka.client.serviceUrl.defaultZone=http://localhost:9000/eureka/
                            eureka.client.initialInstanceInfoReplicationIntervalSeconds=5
                            eureka.client.registryFetchIntervalSeconds=5
                            eureka.instance.leaseRenewalIntervalInSeconds=5
                            eureka.instance.leaseExpirationDurationInSeconds=5
                    
                    in.bta:bta-gateway
                        dependencies
                            ++ org.springframework.cloud:spring-cloud-starter-bootstrap
                            ++ org.springframework.cloud:spring-cloud-config-client

                        configuaration - bootstrap.properties
                            spring.cloud.config.name=gateway
                            spring.cloud.config.discovery.service-id=config
                            spring.cloud.config.discovery.enabled=true
                            
                            eureka.client.serviceUrl.defaultZone=http://localhost:9000/eureka/                    
                    
                    in.bta:bta-profiles
                        dependencies
                            ++ org.springframework.cloud:spring-cloud-starter-bootstrap
                            ++ org.springframework.cloud:spring-cloud-config-client

                        configuaration - bootstrap.properties
                            spring.cloud.config.name=profiles
                            spring.cloud.config.discovery.service-id=config
                            spring.cloud.config.discovery.enabled=true
                            
                            eureka.client.serviceUrl.defaultZone=http://localhost:9000/eureka/   

                    in.bta:bta-txns
                        dependencies
                            ++ org.springframework.cloud:spring-cloud-starter-bootstrap
                            ++ org.springframework.cloud:spring-cloud-config-client

                        configuaration - bootstrap.properties
                            spring.cloud.config.name=txns
                            spring.cloud.config.discovery.service-id=config
                            spring.cloud.config.discovery.enabled=true
                            
                            eureka.client.serviceUrl.defaultZone=http://localhost:9000/eureka/   

                    in.bta:bta-statement
                        dependencies
                            ++ org.springframework.cloud:spring-cloud-starter-bootstrap
                            ++ org.springframework.cloud:spring-cloud-config-client

                        configuaration - bootstrap.properties
                            spring.cloud.config.name=statement
                            spring.cloud.config.discovery.service-id=config
                            spring.cloud.config.discovery.enabled=true
                            
                            eureka.client.serviceUrl.defaultZone=http://localhost:9000/eureka/   


    Assigment-CaseStudy - ERP System
    -------------------------------------------------------------------

        1. To create, update and retrive Employees
        2. To record the Employee TimeSheet,
            a. the timesheet must be recorded date wise - no of hours worked
            b. assuming 8 hours is the standard working hours.
            c. no of hours worked is zero if employee is on leave.
        3. To gneerate the monthly pay slip of an employee where
            a. the hra is 12% of the basic
            b. the ta is 4% of the basic
            c. the leave duductions as per no of hours not worked for.
            d. the gross and net salary

        employees microservice
            Employee
                Long empId
                String fullName
                Double basic

        timesheet microservice
            Employee
                Long empId
                Set<TimeSheet> timeSheets
            TimeSheet
                Long tsId
                LocalDate tsDate
                Double workingHours
                Employee owner

        payslip microservice
            PaySlip
                Long empId
                String fullName
                Double basic
                Month month
                Integer year
                Double hra
                Double ta
                Double deductions
                ...etc.,

    