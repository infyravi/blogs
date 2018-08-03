---
layout: post
title: Loading Resource Bundles Using Wild Card Notation In Spring
date: 2018-07-30 13:32:20 +0300
description: In this post we will see how we can load multiple properties files using wild card notation in Spring framework. It will not only free the development team from specifying the entries in basename each time they add a new properties file but it will also simplify the Spring configuration file by specifying the generic wild card pattern instead of specifying multiple basename entires.
# Add post description (optional)
img: spring.jpg # Add image post (optional)
imgPost: springpost.jpg #Add post header image (optional)
fig-caption: # Add figcaption (optional)
tags: [Spring, resourceBundlers, resources, properties, wildcard,ReloadableResourceBundleMessageSource]
---
In this blog post we will see how we can load multiple resource bundles using wild card notation in Spring framework.The normal way to load the resource bundles is by specifying them in <b><i>basenames </i> </b>property in spring configuration file.This approach suffice when we are having limited resource bundles viz. 1 or 2. Take a scenario, where we have to define multiple resource bundles as the development team stores configuration details in resource bundles for dynamic lookups. Though the team can define all resource bundles in basenames and continue working but over a period of time the following issues can be seen:-<br>
 <ui><li><i>Lenghty configuration file which is difficult to maintain </i></li>
 <li><i>Team need to ensure that all resource bundles are mentioned properly in baseNames</i> </li></ui>
 
 Can there be a solution approach <img src="{{site.baseurl}}/assets/img/thinking.jpg" width="30" height="30"/>through which we can define a generic basename entry in configuration file and it can load all the resource bundles ? This approach will not only keep the configuration file simple but will also free the development team from making any entry in the configuration file.
 
 When I was looking for such an approach,I hit upon this <a href="https://jira.spring.io/secure/attachment/15313/WildcardReloadeableResourceBundleMessageSource.java">jira link</a> which shows how we can subclass from <b> ReloadableResourceBundleMessageSource </b> and build a wild card message source. This is what I was looking for and taking that class as the base i improvised on top of it. 
 
 Using this class and specifying the wild card pattern in Spring configuration file, I was able to load any number of resource bundles.
 
 Enough Gyan given and now its time to see the code in Action!!!!
 <hr>
 
 <b>Technologies used </b>
 <ui>
 <li>Spring 3.2.3 RELEASE</li>
 <li>Java 7</li>
 <li>Jboss Application Server 7.1.0</li>
 </ui>
 
 Following are the set of files which are covered in this blog
 <ui>
 <li><b>WildcardReloadableResourceBundleMessageSource.java</b><br> &emsp;&emsp;&emsp;This class reads the wild card pattern for basenames</li>
 <li><b>Spring XML configuration file</b><br> &emsp;&emsp;&emsp; Spring configuration file where we define the resource bean class along with properties</li>
 <li><b>ResourceBundlesScanner.java</b><br> &emsp;&emsp;&emsp;Utility class which is used to read the values from resource bundle</li>
 <li><b>Test.java</b><br> &emsp;&emsp;&emsp;Test class which shows the working code in action</li>
 </ui>
<hr>

<b><font color="blue">WildcardReloadableResourceBundleMessageSource.java</font> </b>
{% highlight java %}
package org.company.common.utils;

import java.io.IOException;
import java.util.ArrayList;
import java.util.List;

import org.apache.commons.lang.StringUtils;
import org.apache.logging.log4j.LogManager;
import org.apache.logging.log4j.Logger;
import org.springframework.context.support.ReloadableResourceBundleMessageSource;
import org.springframework.core.io.ClassPathResource;
import org.springframework.core.io.FileSystemResource;
import org.springframework.core.io.Resource;
import org.springframework.core.io.UrlResource;
import org.springframework.core.io.VfsResource;
import org.springframework.core.io.support.PathMatchingResourcePatternResolver;
import org.springframework.core.io.support.ResourcePatternResolver;

/**
 * WildcardReloadableResourceBundleMessageSource.java
 * 
 * Spring-specific {@link org.springframework.context.MessageSource}
 * implementation that accesses resource bundles using specified basenames using
 * wild card entries, participating in the Spring
 * {@link org.springframework.context.ApplicationContext}'s resource loading.
 * 
 * <p>
 * This class extends spring's ReloadableResourceBundleMessageSource class and
 * overrides the setBasenames() method by providing the custom implementation to
 * load all the basenames matching the wild card entry specified in the spring
 * configuration file.
 * 
 * <p>
 * It internally uses spring's PathMatchingResourcePatternResolver class to
 * match all the resources matching the specified wild card entry. To use this
 * class we need to replace ReloadableResourceBundleMessageSource in the spring
 * coniguration file with this class. All other features which are supported by
 * ReloadableResourceBundleMessageSource remain as it is.
 * 
 */
public class WildcardReloadableResourceBundleMessageSource extends ReloadableResourceBundleMessageSource {
	
	private static final Logger LOGGER = LogManager.getLogger(WildcardReloadableResourceBundleMessageSource.class.getName());
	/*
	 * Constant to define the properties file extension
	 */
	private static final String PROPERTIESSUFFIX = ".properties";
	/*
	 * Constant to define the classes string in the URI
	 */
	private static final String CLASSES ="/classes/";
	/*
	 * constant to define the class path string
	 */
	private static final String CLASSPATH ="classpath:";
    /*
     * Variable reference to PathMatching resource
     */
	private ResourcePatternResolver resourcePatternResolver = new PathMatchingResourcePatternResolver();


    @Override
    public void setBasenames(String... passedInBaseNames) {
        if (passedInBaseNames != null) {
            List<String> baseNames = new ArrayList<>();
            for (int i = 0, len=passedInBaseNames.length; i < len; i++) {
                String basename = StringUtils.trimToEmpty(passedInBaseNames[i]);
                if (StringUtils.isNotBlank(basename)) {
                    try {
                        Resource[] resources = resourcePatternResolver.getResources(basename);
                        for (int j = 0, resLen=resources.length; j <resLen  ; j++) {
                            Resource resource = resources[j];
                            String baseName = getBaseName(resource);
                            if (baseName != null) {
                                baseNames.add(baseName);
                            }
                        }
                    } catch (IOException e) {
                        logger.error("No message source files found for basename " + basename + ".");
                        logger.error(e);
                    }
                }
                 super.setBasenames(baseNames.toArray(new String[baseNames.size()]));
            }
        }
    }
    
    /**
     * This method is used to get the basename from the resource URI.
     * @param resource
     * @return the baseName
     * @throws IOException
     */
	private String getBaseName(Resource resource) throws IOException {
		String baseName = null;
		String uri = resource.getURI().toString();
		if(!uri.endsWith(PROPERTIESSUFFIX)){
			return baseName;
		}
		/*
		 * On jBOSS application server the property files are stored in virtual
		 * file system so the resource type wil be VfsResource. On local it will
		 * be FileSystemResource. We are using OR condition so that it will work
		 * on both local and jBOSS without any issues
		 */
		if (resource instanceof FileSystemResource || resource instanceof VfsResource) {
			baseName = CLASSPATH + StringUtils.substringBetween(uri, CLASSES, PROPERTIESSUFFIX);
		} else if (resource instanceof ClassPathResource) {
			baseName = StringUtils.substringBefore(uri, PROPERTIESSUFFIX);
		} else if (resource instanceof UrlResource) {
			baseName = CLASSPATH + StringUtils.substringBetween(uri, ".jar!/",PROPERTIESSUFFIX);
		} else {
			LOGGER.info("URI not matched any of the resource types..so just reading it as a string");
			baseName = CLASSPATH + StringUtils.substringBetween(uri, CLASSES,PROPERTIESSUFFIX);
		}
		return baseName;
	}
}

{% endhighlight %}

<b><font color="blue">Spring Configuration File</font></b>

{% highlight xml %}
 <?xml version="1.0" encoding="UTF-8"?>
 <beans xmlns="http://www.springframework.org/schema/beans" 
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
	http://www.springframework.org/schema/beans/spring-beans.xsd">

	<bean id="resourceBundleScanner"
		  class="org.company.common.utils.ResourceBundlesScanner">
		<property name="reloadableResourceBundles" ref="reloadableResourceBundles" />
	</bean>

	<bean id="reloadableResourceBundles"
		class="org.company.common.utils.WildcardReloadableResourceBundleMessageSource">
		<property name="basenames" value="classpath:/messageSources/**/*.*"/>
		<property name="defaultEncoding" value="UTF-8" />
	</bean>
 </beans>
{% endhighlight %}

Please check the wild card pattern <b><font color="#800000"> "/messageSources/**/*.*"</font></b>mentioned in the basenames. It is the CRUX of this blog.

If you notice in my project structure screen shot (shown below <img src="{{site.baseurl}}/assets/img/downicon.png" width="30" height="30"/>) messageSources is the top folder which contains all resource bundles.So you need to change it accordingly as per your project structure.<br>
Now lets understand what the above wild card pattern means.<i><b> It means read all the resource bundles in the messageSources folder along with all resource bundles in the sub folders within it. It can go to any depth.</b></i>

<b><font color="blue">Bean class (ResourceBundlesScanner.java ) which is used to fetch the property values</font></b>

{% highlight java %}
package org.company.common.utils;

import org.apache.logging.log4j.LogManager;
import org.apache.logging.log4j.Logger;

/**
 * This class is used to resolve the given property from the message resource bundle which gets refreshed for a 
 * specific interval.The refreshed properties will be always read and returned.
 * 
 * @author ravi Shankar anupindi
 *
 */

import org.springframework.context.support.ReloadableResourceBundleMessageSource;
import org.springframework.stereotype.Component;

import com.cisco.mbrng.exception.RootException;
import com.cisco.mbrng.exception.ExceptionMessage;

@Component
public class ResourceBundlesScanner {

    private static final Logger LOGGER = LogManager
            .getLogger(ResourceBundlesScanner.class.getName());
    /**
     * Reference of this resource, is used to read the reloaded properties
     */
    private static ReloadableResourceBundleMessageSource reloadableResourceBundles;

    public static ReloadableResourceBundleMessageSource getReloadableResourceBundles() {
        return reloadableResourceBundles;
    }

    /**
     * Setter based message resource injection
     * 
     * @param reloadableResourceBundles
     *            - resource which is injected
     */
    public static void setReloadableResourceBundles(
            final ReloadableResourceBundleMessageSource reloadableResourceBundles) {
        ResourceBundlesScanner.reloadableResourceBundles = reloadableResourceBundles;
    }

    /**
     * Read the property value from the resource
     * 
     * @param key
     *            - given key
     * @return - value, set to the property
     */ 
    public static String getMessage(final String key) {
        String retVal = "";
        try {
            retVal = reloadableResourceBundles.getMessage(key, null, null);
        } catch (Exception exception) {
            LOGGER.debug("Resource bundle key definition is missing - "+key,
                    exception);
            return null;
        }
        return retVal;
    }
    
	 
	/**
	 * Read the property value from the resource. if the value is not found then
	 * the passed in default value will be returned instead.
	 * 
	 * @param key
	 *            - given key
	 * @param defaultValue
	 *            - given value to return if the property is not found
	 * 
	 * @return - value, set to the property
	 */
    public static String getMessageOrDefault(final String key, final String defaultValue) {
    	return reloadableResourceBundles.getMessage(key, null, defaultValue, null);
    }
}
{% endhighlight %}

<b> <font color="blue">Multiple Resource Bundles</font> </b>
 This is just a glimpse of how the multiple resource bundles structure will look like
 <p>
 <img src="{{site.baseurl}}/assets/img/projectstructure.jpg"/>
 </p> with following content in each resource bundle
 <div class="blog-div"> 
<b>prop.properties</b>
<p>
&emsp;NAME=Anupindi Ravi Shankar
</p>
<b>prop2.properties</b>
<p>
&emsp;ID=1234
</p>

<b>prop3.properties</b>
<p>
&emsp;ROLE=Software Architect</p>
</div>
<b><font color="blue">Test.java</font></b><br>
 This file shows how we can use the ResourceBundlesScanner to read the messages by specifying the 'key'
{% highlight java %}
package org.company.main;

import org.company.common.utils.ResourceBundlesScanner;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class Test {

	@Autowired
	ResourceBundlesScanner scanner;

	public static void main(String[] args) {
		ApplicationContext context = new ClassPathXmlApplicationContext(
				"applicationContext.xml");

		Test myTest = new Test();
		myTest.printMessage(context);
	}

	private void printMessage(ApplicationContext context) {
		System.out.println(String.format("Key is %s and its value is %s ",
				"NAME", scanner.getMessage("NAME")));

		// When property is not defined..return the default value
		System.out.println(String.format(
				"Key is %s and its DEFAULT value is %s ", "NAME",
				scanner.getMessageOrDefault("id", "94914")));

		System.out.println(String.format("Key is %s and its value is %s ",
				"ID", scanner.getMessage("ID")));

		System.out.println(String.format("Key is %s and its value is %s ",
				"ROLE", scanner.getMessage("ROLE")));
	}
}
{% endhighlight %}
Following is the <b>output</b> which gets printed on the console
<div class="output-div">
&emsp;Key is <b>NAME</b> and its value is <b>Anupindi Ravi Shankar</b><br>
&emsp;Key is <b>NAME</b> and its DEFAULT value is <b>94914</b> <br>
&emsp;Key is <b>ID</b> and its value is <b>1234</b> <br>
&emsp;Key is <b>ROLE</b> and its value is <b>Software Architect</b> <br>
</div>

The complete working code can be downloaded from my <a href="https://github.com/infyravi/resource-bundles">GitHub repository </a>
<p>
<b> Conclusion </b> <p>
So in this blog we have seen how we can use wild card pattern to load multiple resource bundles using Spring framework.
</p>
Please leave your comments !!!  Also you can subscrbe to the Atom feed.