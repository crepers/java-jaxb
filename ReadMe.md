# Java code from JAXB with maven plugin
Web Services를 사용하기 위해 stub class들을 만들어야 합니다. xjc command 뿐만 아니라 maven을 이용하여 stub class를 생성할 수 있습니다.

## xjc command
1. XJC(JAXB Binding Complier)
  - xjc 명령어를 사용하여 java class를 생성할 수 있습니다.
  - Serializable이 설정이 별도로 필요합니다.
  - Command
    ```
    xjc -d output -p com.wcjung.sample -encoding UTF-8 *
    xjc -d output -p com.wcjung.sample -encoding UTF-8 -wsdl http://10.xxx.xxx.xxx/services/
    xjc -d output -p com.wcjung.sample -encoding UTF-8 -wsdl http://10.xxx.xxx.xxx/services/ServicePort?wsdl
    ```

---

## Maven
### 1. Maven plugin
maven-compiler-plugin 을 이용하여 java class를 생성할 수 있습니다.

### 2. pom.xml 
    - maven-compiler-plugin을 구성합니다.
    - java build version을 설정합니다.

```xml
    <pluginManagement>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                </configuration>
            </plugin>
            ...
        </plugins>
    </pluginManagement>
```

    - maven-jaxb2-plugin 설정합니다.
    - encoding : encoding 정보(UTF-8)
    - schemaDirectory : WSDL 또는 XSD가 있는 디렉토리
    - generatePackage : package 정보

```xml
    <pluginManagement>
        <plugins>
            ...
            <plugin>
                <groupId>org.jvnet.jaxb2.maven2</groupId>
                <artifactId>maven-jaxb2-plugin</artifactId>
                <version>0.13.1</version>
                <executions>
                    <execution>
                        <goals>
                            <goal>generate</goal>
                        </goals>
                    </execution>
                </executions>
                <configuration>
                    <encoding>UTF-8</encoding>
                    <schemaDirectory>${project.basedir}/src/main/webapp/wsdl</schemaDirectory>
                    <generatePackage>com.wcjung.sample.ws</generatePackage>
                    <schemaIncludes>
                        <include>*.wsdl</include>
                    </schemaIncludes>
                </configuration>
            </plugin>
        </plugins>
    </pluginManagement>
```

### 3. 결과
- 디렉토리 : project.basedir\target\generated-sources\xjc
- package 별로 java class가 생성됩니다.

---
## Serialize
  직렬화가 필요한 경우 Serializable Interface를 상속받아 
구현해야만 합니다.
### 1. pom.xml
- bindingDirectory : 바인딩할 파일이 있는 디렉토리
- bindingInclude : 바인딩 파일
    
    ```xml
    <build>
        ...
        <plugins>
            <plugin>
                ...
                <configuration>
                    ...
                    <bindingDirectory>${project.basedir}/src/main/webapp/wsdlBinding</bindingDirectory>
                        <bindingIncludes>						
                            <bindingInclude>binding_serialize.xjb</bindingInclude>
                        </bindingIncludes>
                </configuration>
            </plugin>
        </plugins>
    </build>
    ```

### 2. binding file
- binding_serialize.xjb
```xml
<jaxb:bindings version="1.0" xmlns:jaxb="http://java.sun.com/xml/ns/jaxb"
               xmlns:xsd="http://www.w3.org/2001/XMLSchema"
               xmlns:xjc="http://java.sun.com/xml/ns/jaxb/xjc">
    <jaxb:globalBindings>
        <jaxb:serializable uid="-1"/>
    </jaxb:globalBindings>
</jaxb:bindings>
```

### 3. ouput class
```java
public class test implements Serializable {
    private final static long serialVersionUID = -1L;
    ...
}
```

---

## Date Type
  JAXB 기본 데이터 타입인 XMLGregorianCalendar type을 Serializable type으로 변경이 필요한경우 XMLGregorianCalendar type은 직렬화를 지원하지 않기 때문에 직렬화가 가능한 GregorianCalendar로 변경이 필요합니다.

- pom.xml
    ```xml
    <build>
        ...
        <plugins>
            <plugin>
                ...
                <configuration>
                    ...
                    <bindingDirectory>${project.basedir}/src/main/webapp/wsdlBinding</bindingDirectory>
                        <bindingIncludes>						
                            <bindingInclude>binding_dateType.xjb</bindingInclude>
                        </bindingIncludes>
                </configuration>
            </plugin>
        </plugins>
    </build>
    ```
- binding_dateType.xjb
    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <jaxb:bindings version="1.0" xmlns:jaxb="http://java.sun.com/xml/ns/jaxb"
                xmlns:xsd="http://www.w3.org/2001/XMLSchema"
                xmlns:xjc="http://java.sun.com/xml/ns/jaxb/xjc">
        <jaxb:globalBindings>
            <jaxb:serializable uid="-1"/>
            <jaxb:javaType name="java.util.GregorianCalendar"
                xmlType="xsd:dateTime"
                parseMethod="javax.xml.bind.DatatypeConverter.parseDateTime"
                printMethod="javax.xml.bind.DatatypeConverter.printDateTime" />
            <jaxb:javaType name="java.util.GregorianCalendar"
                xmlType="xsd:date"
                parseMethod="javax.xml.bind.DatatypeConverter.parseDate"
                printMethod="javax.xml.bind.DatatypeConverter.printDate" />
            <jaxb:javaType name="java.util.GregorianCalendar"
                xmlType="xsd:time"
                parseMethod="javax.xml.bind.DatatypeConverter.parseTime"
                printMethod="javax.xml.bind.DatatypeConverter.printTime" />
        </jaxb:globalBindings>
    </jaxb:bindings>
    ```
