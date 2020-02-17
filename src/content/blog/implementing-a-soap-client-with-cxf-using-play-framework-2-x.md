---
title: "Implementing a SOAP client with CXF using Play Framework 2.x"
date: "2015-12-21"
draft: false
tags: [Java, Play, sbt, SOAP, CXF, WSDL, Guice, wsdl2java, Spring]
featured: true
---

**TL;DR: if you want to skip the tutorial, all code is directly available on GitHub: [https://github.com/damienbeaufils/soap-client-with-cxf-using-play](https://github.com/damienbeaufils/soap-client-with-cxf-using-play)** 

**EDIT June 2016: code on my GitHub repository updated for Play Framework 2.5! Play 2.4 version is still available [here](https://github.com/damienbeaufils/soap-client-with-cxf-using-play/tree/play-2.4.x).**

I was working on a new client product, and I had to plug a web application built with Play Framework (and then sbt) to SOAP web services. How to do it?

The need: be able to generate Java classes from one or many WSDL files, and use those classes with Play 2.4 dependency injection (which use Guice as implementation).

The issue: no _out-of-the-box_ SOAP support on Play.

The good news: there is an official plugin from Typesafe which answer the need: [play-soap-sbt](http://downloads.typesafe.com/rp/play-soap/Home.html). It promises a reactive implementation of SOAP web services calls from your application.

The bad news: this plugin is a part of the _Typesafe Reactive Plateform_ suite, and you have to pay an unknown amount (I didn't found any value on the website) to be able to use this plugin in your application.

Coming from the Java/Groovy world, I often used Spring framework. When I had to use SOAP in an application, I used CXF most of the time, which works perfectly with Spring. So I asked myself **how to use CXF with Play?**

I didn't found any miracle answer on the web, so I built a custom solution, inspired mainly by [play-cxf](https://github.com/imindeu/play-cxf/tree/play-2.4.x). So here is a tutorial, step by step, to implement a SOAP client with CXF 3.1.x using Play Framework 2.4.x:

#### Step 1: create a new Play application with activator

Optional step if you already have an existing application.

```
# cf. https://www.playframework.com/documentation/2.4.x/NewApplication
activator new soap-client-with-cxf-using-play play-java
```

#### Step 2: download a WSDL and store it in the project

I use for this tutorial the [GlobalWeather](http://www.webservicex.net/globalweather.asmx) SOAP web service. Once the [WSDL](http://www.webservicex.net/globalweather.asmx?wsdl) downloaded, the file is saved in the **conf/wsdls** folder:

```
cd soap-client-with-cxf-using-play/conf
mkdir wsdls
wget http://www.webservicex.net/globalweather.asmx?WSDL -O wsdls/globalweather.wsdl
```

#### Step 3: add the [sbt-cxf-wsdl2java](https://github.com/ebiznext/sbt-cxf-wsdl2java) plugin

In the **project/plugins.sbt** file:

```
resolvers += "Sonatype Repository" at "https://oss.sonatype.org/content/groups/public"
addSbtPlugin("com.ebiznext.sbt.plugins" % "sbt-cxf-wsdl2java" % "0.1.4")
```

#### Step 4: add CXF required dependencies

In the **build.sbt** file:

```
val cxfVersion: String = "3.1.4"
 
libraryDependencies ++= Seq(
  ...
  "org.apache.cxf" % "cxf-rt-frontend-jaxws" % cxfVersion,
  "org.apache.cxf" % "cxf-rt-transports-http" % cxfVersion
)
```

#### Step 5: configure the wsdl2java task to automatically generate the classes corresponding to the downloaded WSDL

In the **build.sbt** file:

```
// CXF wsdl2java configuration
Seq(cxf.settings: _*)
cxf.cxfVersion := cxfVersion
cxf.wsdls := Seq(
  cxf.Wsdl((resourceDirectory in Compile).value / "wsdls/globalweather.wsdl", Seq("-mark-generated", "-p", "com.global.weather"), "globalweather")
)
```

#### Step 6: launch wsdl2java task

```
activator clean wsdl2java
```

or

```
sbt clean wsdl2java
```

Generated classes should then be found in **target/cxf/globalweather** folder:

```
tree target/cxf/globalweather
 
target/cxf/globalweather/
└── com
    └── global
        └── weather
            ├── GetCitiesByCountry.java
            ├── GetCitiesByCountryResponse.java
            ├── GetWeather.java
            ├── GetWeatherResponse.java
            ├── GlobalWeatherHttpGet_GlobalWeatherHttpGet_Client.java
            ├── GlobalWeatherHttpGet.java
            ├── GlobalWeatherHttpPost_GlobalWeatherHttpPost_Client.java
            ├── GlobalWeatherHttpPost.java
            ├── GlobalWeather.java
            ├── GlobalWeatherSoap_GlobalWeatherSoap12_Client.java
            ├── GlobalWeatherSoap_GlobalWeatherSoap_Client.java
            ├── GlobalWeatherSoap.java
            ├── ObjectFactory.java
            └── package-info.java
 
3 directories, 14 files
```

#### Step 7: add Spring Context dependency

Generated classes are now in the classpath, but we have to use Spring to be able to instanciate and use CXF JAX-WS client.

```
libraryDependencies ++= Seq(
  ...
  "org.springframework" % "spring-context" % "4.2.4.RELEASE"
)
```

#### Step 8: declare CXF generated JAX-WS client in the Spring context

In the CXF generated sources, an interface with [@WebService](https://docs.oracle.com/javaee/5/api/javax/jws/WebService.html) annotation has been created and we can use it to easily declare the JAX-WS client. In this example, Spring context file is named **applicationContext.xml** and stored in **conf** folder:

```
<beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:jaxws="http://cxf.apache.org/jaxws" xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.2.xsd http://cxf.apache.org/jaxws http://cxf.apache.org/schemas/jaxws.xsd">
 
    <import resource="classpath*:META-INF/cxf/cxf.xml"/>
 
    <jaxws:client id="globalWeatherSoapClient" serviceClass="com.global.weather.GlobalWeatherSoap" address="${global.weather.host}/globalweather.asmx"/>
 
</beans>
```

#### Step 9: configure Play to load Spring context when application starts

For this, it is necessary to create a module, which will programmatically load Spring context, and to declare it in application configuration.

In this example, module is named **ApplicationContextBinderModule** and stored in **app/modules** folder:

```
package modules;
 
import com.google.inject.AbstractModule;
import org.springframework.context.support.ClassPathXmlApplicationContext;
 
public class ApplicationContextBinderModule extends AbstractModule {
 
    @Override
    protected void configure() {
        ClassPathXmlApplicationContext applicationContext = new ClassPathXmlApplicationContext("applicationContext.xml");
 
        applicationContext.start();
        applicationContext.registerShutdownHook();
    }
 
}
```

The module is then declared in application configuration. Here in the main configuration file **application.conf**:

```
play.modules.enabled += "modules.ApplicationContextBinderModule"
```

#### Step 10: configure Play dependency injection (Guice) to be able to load the JAX-WS client with the @Inject annotation

With the Spring context loaded, a bean globalWeatherSoapClient exists, so we have now [to tell to Guice which instance to bind](https://github.com/google/guice/wiki/InstanceBindings) when we want to inject a GlobalWeatherSoap dependency:

```
package modules;
 
import com.global.weather.GlobalWeatherSoap;
import com.google.inject.AbstractModule;
import org.springframework.context.support.ClassPathXmlApplicationContext;
 
public class ApplicationContextBinderModule extends AbstractModule {
 
    @Override
    protected void configure() {
        ClassPathXmlApplicationContext applicationContext = new ClassPathXmlApplicationContext("applicationContext.xml");
 
        applicationContext.start();
        applicationContext.registerShutdownHook();
 
        // Guice instance binding
        bind(GlobalWeatherSoap.class).toInstance((GlobalWeatherSoap) applicationContext.getBean("globalWeatherSoapClient"));
    }
 
}
```

#### Step 11: inject the JAX-WS client to call SOAP web services

Example in a **GlobalWeatherController** controller:

```
# Routes
# ~~~~
GET /cities/:country @controllers.GlobalWeatherController.getCities(country: String)
GET /weather/:country/:city @controllers.GlobalWeatherController.getWeather(country: String, city: String)
```

```
package controllers;
 
import com.global.weather.GlobalWeatherSoap;
import play.mvc.Controller;
import play.mvc.Result;
 
import javax.inject.Inject;
 
public class GlobalWeatherController extends Controller {
 
    @Inject
    private GlobalWeatherSoap globalWeatherSoapClient;
 
    public Result getCities(String countryName) {
        String cities = globalWeatherSoapClient.getCitiesByCountry(countryName);
        return ok(cities).as("text/xml");
    }
 
    public Result getWeather(String countryName, String cityName) {
        String weather = globalWeatherSoapClient.getWeather(cityName, countryName);
        return ok(weather).as("text/xml");
    }
}
```

#### Step 12: run and test!

```
activator run
```

or

```
sbt run
```

and then

```
curl http://localhost:9000/cities/France
```

or

```
curl http://localhost:9000/weather/France/Cognac
```

Second request result:

```
<CurrentWeather>
    <Location>Cognac, France (LFBG) 45-40N 000-19W 31M</Location>
    <Time>Nov 09, 2015 - 07:30 AM EST / 2015.11.09 1230 UTC</Time>
    <Wind> from the SSW (200 degrees) at 7 MPH (6 KT):0</Wind>
    <Visibility> greater than 7 mile(s):0</Visibility>
    <SkyConditions> overcast</SkyConditions>
    <Temperature> 64 F (18 C)</Temperature>
    <DewPoint> 59 F (15 C)</DewPoint>
    <RelativeHumidity> 82%</RelativeHumidity>
    <Pressure> 30.47 in. Hg (1032 hPa)</Pressure>
    <Status>Success</Status>
</CurrentWeather>
```

#### Step 13 (bonus): use Play configuration in Spring context

The applicationContext.xml file we use contains only hard-coded values. When we use SOAP web services, we often have several environments (dev vs. prod).

It is possible to load configuration files in Spring context, with PropertySourcesPlaceholderConfigurer, but it is designed to read properties files and not HOCON files, which are used by Play Framework.

So I created a **HoconPropertySourcesPlaceholderConfigurer** class, which is able to read a \*.conf file in order to use it in Spring context with SpEL.

```
package utils;
 
import org.apache.commons.lang3.StringUtils;
import org.springframework.beans.BeansException;
import org.springframework.beans.factory.config.ConfigurableListableBeanFactory;
import org.springframework.context.support.PropertySourcesPlaceholderConfigurer;
import org.springframework.core.env.ConfigurablePropertyResolver;
import org.springframework.util.StringValueResolver;
 
public class HoconPropertySourcesPlaceholderConfigurer extends PropertySourcesPlaceholderConfigurer {
 
    @Override
    protected void processProperties(ConfigurableListableBeanFactory beanFactoryToProcess, ConfigurablePropertyResolver propertyResolver) throws BeansException {
        propertyResolver.setPlaceholderPrefix(this.placeholderPrefix);
        propertyResolver.setPlaceholderSuffix(this.placeholderSuffix);
        propertyResolver.setValueSeparator(this.valueSeparator);
 
        StringValueResolver valueResolver = strVal -> {
            String resolved = ignoreUnresolvablePlaceholders ?
                    propertyResolver.resolvePlaceholders(strVal) :
                    propertyResolver.resolveRequiredPlaceholders(strVal);
            return resolved.equals(nullValue) ? null : StringUtils.replace(resolved, "\"", "");
        };
 
        doProcessProperties(beanFactoryToProcess, valueResolver);
    }
}
```

The applicationContext.xml file then become:

```
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:jaxws="http://cxf.apache.org/jaxws"
       xmlns:http-conf="http://cxf.apache.org/transports/http/configuration"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.2.xsd
        http://cxf.apache.org/jaxws http://cxf.apache.org/schemas/jaxws.xsd
        http://cxf.apache.org/transports/http/configuration http://cxf.apache.org/schemas/configuration/http-conf.xsd">
 
    <import resource="classpath*:META-INF/cxf/cxf.xml"/>
 
    <bean class="utils.HoconPropertySourcesPlaceholderConfigurer">
        <!-- Load configuration file defined in config.resource (see https://www.playframework.com/documentation/2.4.x/ProductionConfiguration) -->
        <!-- If no config.resource defined, use application.conf -->
        <property name="location" value="#{systemProperties['config.resource'] ?: 'application.conf'}"/>
    </bean>
 
    <jaxws:client id="globalWeatherSoapClient"
                  serviceClass="com.global.weather.GlobalWeatherSoap"
                  address="${global.weather.host}/globalweather.asmx"/>
 
    <http-conf:conduit name="${global.weather.host}/.*">
        <http-conf:client ConnectionTimeout="${global.weather.connection.timeout}"
                          ReceiveTimeout="${global.weather.response.timeout}"
                          AllowChunking="false"/>
    </http-conf:conduit>
 
</beans>
```

And in configuration:

```
# Global weather SOAP client configuration
# ~~~~~
global.weather.host = "http://www.webservicex.net"
global.weather.connection.timeout = 60000
global.weather.response.timeout = 60000
```

#### Final code

All the code of this tutorial is available on GitHub: [https://github.com/damienbeaufils/soap-client-with-cxf-using-play](https://github.com/damienbeaufils/soap-client-with-cxf-using-play)

_Feel free to fork & enjoy!_
