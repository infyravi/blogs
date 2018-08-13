---
layout: post
title: "Dynamic Logging with Log4j2"
date: 2018-08-09 13:32:20 +0300
description: We recently came with the requirement to change the application log levels at run time without making any code changes and avoid the code build and deployment steps.Since it is a very common requirement, so I thought to post it in blog so that it can be useful for others . # Add post description (optional)
img: log4j1.jpg # Add image post (optional)
imgPost: log4j.jpg #Add post header image (optional)
tags: [log4j,log4j2,log level, appenders, loggers]
---
I recently came across the requirement to change the application log levels dynamically to debug the issues reported by the QA team. We were following the approach of changing the log level in log4j2.xml file, give the Jenkins build and then do deployment on the required environment.
<p> 
This being a very time consuming approach we were looking for an alternative approach which enables us to change the log level dyancamically without touching the code.
We were using log4j2 as the logging framework in our application. 
After doing some research came across the following 2 approaches which people talked about in different forums
  <ol>
   <li> <b>JMX based approach</b>. We need to add separate JVM argument</li>
   <li> <b>Create a Watcher service</b> which listens for changes to log4j2.xml file. We need to keep the log4j2.xml cnofguration file outside the EAR file.</li>
  </ol>
In our case, both the above approaches didn't work as it needs operations team approval. So we thought of going with a 
Controlled REST based approach. When I say controlled, it means only the ADMIN user can change the log level and we don't have to depend on the operations team. <p>With limited number of JVMs configured triggering LogConfigManager is easy but if you are having many JVMs then you need to think of how to trigger the LogConfigManager for all JVMs.

Since we were using Oracle DB, so we leveraged <b>Oracle Change Notification</b> mechanism to trigger LogConfigManager whenever we changed the log level in the DB. There can be many approaches, so you need to define the approach based on your project which fits the best.

<p>Herewith I am presenting the approach from REST end point perspective and not the one using Change Notifications.

<hr>
<b><font color="blue">End To End Architectural flow</font></b><br>
<br>
<br>
<img src="{{site.baseurl}}/assets/img/loggingArchitectire.png" width="75%" height="300"/>
<br>
Lets understand each component mentioned in the architecture
<p>
1. <b>/loglevel </b> is the REST Endpoint <br>
2. Input will be sent in the JSON format which consist of <i>loggerName</i> and <i>Level</i> attributes.<br>
3. <b>Validation </b>layer enables only the user with ADMIN access to change the log level <br>
4. <b>LogConfigManager</b> is the utility class which changes the log level dynamically.<br>
<hr>
<b><font color="blue">LogConfigManager</font></b><br>
    This class is responsible for changing the log level dynamically.
	
{% highlight java %}

import java.util.Map;
import java.util.Map.Entry;

import org.apache.logging.log4j.Level;
import org.apache.logging.log4j.LogManager;
import org.apache.logging.log4j.core.Appender;
import org.apache.logging.log4j.core.LoggerContext;
import org.apache.logging.log4j.core.config.Configuration;
import org.apache.logging.log4j.core.config.LoggerConfig;


public class LogConfigManager {
	 
 private static final String ALL_LOGGERS ="ALL";
 private static final String DEFAULT_LEVEL ="default";
 private static final String ROOT_LOGGER ="root";
 
 /**
  * This method will set the log level for all loggers and all appenders
  * within each logger.
  * @param level
  * 
  * Levels used for identifying the severity of an event. Levels are organized from most specific to least:
  * <ul>
  * <li>{@link #OFF} (most specific, no logging)</li>
  * <li>{@link #FATAL} (most specific, little data)</li>
  * <li>{@link #ERROR}</li>
  * <li>{@link #WARN}</li>
  * <li>{@link #INFO}</li>
  * <li>{@link #DEBUG}</li>
  * <li>{@link #TRACE} (least specific, a lot of data)</li>
  * <li>{@link #ALL} (least specific, all data)</li>
  * </ul>
  *
  */
 
 public static void setLogLevel(final String level){
	 setLogLevel(ALL_LOGGERS,level, true);
 }

/**
 * It is the overloaded method which accepts the loggername also. It sets
 * the passed in level to the corresponding logger and also to all appenders
 * within that logger
 * 
 * @param loggerName
 * @param level
 */
 public static void setLogLevel(final String loggerName,final String level){
	 setLogLevel(loggerName,level, true);
 }
 
 /**
  * This is the actual method which set the log levels accordingly.
  * If the loggerName is passed as "ALL" or it is NULL then it will be applicable
  * for all loggers.
  * If the passedInLevel is null then the default level "debug" will be considered
  * 
  * @param loggerName
  * @param passedInLevel
  * @param changeAppenders
  */
  public static void setLogLevel(final String loggerName,final String passedInLevel,final boolean changeAppenders)
  {
	 boolean allLoggers= (loggerName==null || ALL_LOGGERS.equals(loggerName.toUpperCase()))?true:false;
	 String applyLevel = passedInLevel!=null? passedInLevel : DEFAULT_LEVEL;

	 Level newLevel = Level.valueOf(applyLevel);  	
	 LoggerContext ctx = (LoggerContext) LogManager.getContext(false);
	 Configuration config = ctx.getConfiguration();
	/**
	* Apply for all LOGGERS
	*/
	if(allLoggers){
	 Map<String, LoggerConfig> loggers = config.getLoggers();
	 for (Entry<String, LoggerConfig> logger : loggers.entrySet()){ 
		 LoggerConfig  loggerConfig =(LoggerConfig)logger.getValue();
		 loggerConfig.setLevel(newLevel);
		 if(changeAppenders){
			 applyForAppenders(config,loggerConfig,newLevel);
		 }
	  }
	 }else {
	/**
	* Apply only for the passed in Logger name
	*/
	 String newLoggerName = loggerName.equalsIgnoreCase(ROOT_LOGGER)?LogManager.ROOT_LOGGER_NAME:loggerName;
	 LoggerConfig loggerConfig = config.getLoggerConfig(newLoggerName);
	 if (loggerConfig.getName().equalsIgnoreCase(newLoggerName)) {
		 loggerConfig.setLevel(newLevel);
		 if(applyForAppenders){
			 applyForAppenders(config,loggerConfig,newLevel);
		 }
	 }
 }

 ctx.updateLoggers();

 }

 ///////////////////////////////////PRIVATE METHOD////////////////////////////////////////////
/**
* This method is used to set the log level for all the appenders for each logger.
* @param config
* @param loggerConfig
* @param newLevel
*/
 private static void applyForAppenders(Configuration config,LoggerConfig loggerConfig,Level newLevel){
	 Map<String, Appender> appenders = loggerConfig.getAppenders();
	 for (Entry<String, Appender> entry : appenders.entrySet()){ 
		 Appender  app =(Appender)entry.getValue();
		 loggerConfig.removeAppender(app.getName());
		 loggerConfig.addAppender(config.getAppender(app.getName()), newLevel, null);
	 }
 }
}
{% endhighlight %}

In the above class 	<i><b>setLogLevel</b> </i> is the method which does the trick of changing the log levels dynamically.
It uses the following approach:-
<br>
 1.)  It will check if we need to change the log levels of ALL LOGGERS or only to the particular logger for which the name is passed.
      If the loggername is not passed or ALL is passed then it will change the level of all loggers with the passed in level value.<br>
 2.)  From the value of <i> allLoggers </i> flag it decides if it need to change the log level for all the loggers or not.<br>
 3.)  <b>changeAppenders </b>flag will determine if the level need to be changed for all the appenders which the logger refers to.<br>

 <hr>
<b><font color="blue">REST End Point</font></b><br>
 Below is the REST End Point which takes following parameters as input 
 <ui><li><b>loggerName</b> : Name of the logger for which level need to be changed. Default to 'ROOT' </li>
 <li><b>level</b> : Log level which need to be set. Default to 'DEBUG'</li>
 It internally calls <i>LogConfigManager</i> which set the log level for the passed in logger accordingly.
 
{% highlight java %}
   @POST
   @Path("/loglevel")
   @Consumes(MediaType.APPLICATION_JSON)
   @Produces(MediaType.APPLICATION_JSON)
   public Response changeLogLevel(InputStream stream) {
     if(stream!=null && stream.available()){
	  try {
		ObjectMapper mapper = new ObjectMapper();
		JsonNode jsonTree = mapper.readTree(stream);
		String loggerName = jsonTree.get("loggerName").asText();
		String level = jsonTree.get("level").asText();

		LogConfigManager.setLogLevel(loggerName, level, true);

	 }catch(IOException e){
		LOGGER.error(e);
		sendErrorResponse("{\"status\":\"Log Level changed successfully\"}",Response.status(Status.BAD_REQUEST));
	  }finally {
		sendResponse("{\"status\":\"Log Level changed successfully\"}",Response.status(Status.OK));
	  }
	}
	 return null;
	}
	{% endhighlight %}
	We can add vaidation to check if the logged in user is entitled to change the log level or not.
	
	<hr>
	<b><font color="blue">Send Response Method</font></b><br>
     This method send the response back to the client in the JSON format.
	
	{% highlight java %}
	
	/**
	 * This Method is used to send the response back to client.
	 * 
	 * @param object, responseBuilder
	 * @return Response
	 */
	public Response sendResponse(Object object,ResponseBuilder responseBuilder) {
		responseBuilder.type(MediaType.APPLICATION_JSON);
		responseBuilder.entity(object);
		return responseBuilder.build();
	}
	{% endhighlight %}
	<hr>
	<b><font color="blue">Log4j2.xml Configuration File</font></b><br>
     Sample Lo4j2.xml configuration file 
	{% highlight xml %}
<?xml version="1.0" encoding="UTF-8"?>
<Configuration xmlns="http://logging.apache.org/log4j/2.0/config">
	<Appenders>
		<File name="FILE" fileName="/logs/server.log" 	append="true">
			 <PatternLayout pattern="%d{yyyy-MM-dd HH:mm:ss,SSS}-%t-%x-%-5p-%-10c:%m%n"/>
		</File>
		<Console name="STDOUT" target="SYSTEM_OUT">
			 <PatternLayout pattern="%d{yyyy-MM-dd HH:mm:ss,SSS}-%t-%x-%-5p-%-10c:%m%n"/>
		</Console>
	</Appenders>

	<Loggers>
		<Logger name="appLogger" level="info" additivity="false">
			<appender-ref ref="FILE" level="info" />
		</Logger>
		<Root level="info" additivity="false">
			<AppenderRef ref="STDOUT" level="info" />
		</Root>
	</Loggers>
</Configuration>
	{% endhighlight %}
<p>
<b> Conclusion </b> <p>
So in this blog we have seen how we can change the log level programmatically through REST based approach.
</p>
Please leave your comments !!!  Also you can subscrbe to the Atom feed.