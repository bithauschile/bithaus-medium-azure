Bithaus Medium - Azure Driver
=============================

**This repo adds Azure Service Bus support to [Bithaus Medium](https://github.com/bithauschile/bithaus-medium)**

Bithaus Medium Repo: https://github.com/bithauschile/bithaus-medium

Medium provides an abstraction layer to message broker technologies such as Azure Service Bus and Kafka, so that you can use the same message library across your applications without the need of ugly generated message classes or having to worry about what message broker is handling your event. 
 
 <br>
 
## Quickstart
<b>Simple Message definition</b>

    public class TestMessage extends MediumMessage {

        public String name;
        public Integer age;
    }
    
<b>Using the MessagingService</b>

    Map configMap = ... // Uses Kafka style configuration
    MediumMessagingServiceConfig config = new MediumMessagingServiceConfig(configMap);
    MediumMessagingService instance = new MediumMessagingService(config);

    instance.addMessageListener(TestMessage.class, new MediumMessageListener<TestMessage>() {
        @Override
        public String getName() {
            return "listener-1";
        }

        @Override
        public void onMessage(TestMessage message) throws MediumMessageListenerException {
            
            System.out.println("Name: " + message.name + " Age: " + message.age);
        }
    });
<br>

***
<br>

## Complete examples
__Producer__

    public class SimpleMediumKafkaProducerApp {
        
        private static Boolean running = true ;

        public static void Main(String args[]) throws Exception {

            Logger logger = LoggerFactory.getLogger("ProducerApp");

            Map<K,V> originals = new HashMap<>();
            // First the Messaging Service configuration
            originals.put(MediumMessagingServiceConfig.DRIVER_CLASSNAME_CONFIG, MediumMessagingServiceAzureServiceBusDriver.class.getName());
            originals.put(MediumMessagingServiceConfig.LOGGER_SUFIX_CONFIG, "ServiceBusProducerApp");
            // Now the Azure Service Bus driver configuration
            map.put(MediumMessagingServiceAzureServiceBusDriverConfig.CONNECTIONSTRING_CONFIG, "Endpoint=sb://your-sb.servicebus.windows.net/;SharedAccessKeyName=RootManageSharedAccessKey;SharedAccessKey=YOUR_KEY=");
            map.put(MediumMessagingServiceAzureServiceBusDriverConfig.CREDENTIAL_KEY_NAME_CONFIG, "your-key-name");
            map.put(MediumMessagingServiceAzureServiceBusDriverConfig.CREDENTIAL_KEY_CONFIG, "your-key");
            map.put(MediumMessagingServiceAzureServiceBusDriverConfig.SENDER_ENABLED_CONFIG, "true");
            map.put(MediumMessagingServiceAzureServiceBusDriverConfig.SENDER_QUEUE_CONFIG, "QUEUE_NAME");

            MediumMessagingServiceConfig config = new MediumMessagingServiceConfig(originals);
            MediumMessagingService instance = new MediumMessagingService(config);

            instance.start();

            logger.info("Messaging service started");

            Runtime.getRuntime().addShutdownHook(new Thread() {
                @Override
                public void run() {
                    logger.info("Stopping messaging service");
                    running = false;
                    instance.stop();
                }
            });
    
            while(running) {
                TestMessage message = new TestMessage("Hello World @ " + new Date());
                try {
                    instance.send(message);
                } catch (MediumMessagingServiceException ex) {
                    logger.error("Error sending message", ex);
                    System.exit(1);
                }
                Thread.sleep(1000);
            }

        }
    }


__Consumer__

    public class SimpleMediumKafkaConsumerApp {
        
        public static void Main(String args[]) throws Exception {

            Logger logger = LoggerFactory.getLogger("ConsumerApp");
            Gson gson = new Gson();

            Map<K,V> originals = new HashMap<>();
            // First the Messaging Service configuration
            originals.put(MediumMessagingServiceConfig.DRIVER_CLASSNAME_CONFIG, MediumMessagingServiceAzureServiceBusDriver.class.getName());
            originals.put(MediumMessagingServiceConfig.LOGGER_SUFIX_CONFIG, "ServiceBusConsumerApp");
            // Now the Azure Service Bus driver configuration
            map.put(MediumMessagingServiceAzureServiceBusDriverConfig.CONNECTIONSTRING_CONFIG, "Endpoint=sb://your-sb.servicebus.windows.net/;SharedAccessKeyName=RootManageSharedAccessKey;SharedAccessKey=YOUR_KEY=");
            map.put(MediumMessagingServiceAzureServiceBusDriverConfig.CREDENTIAL_KEY_NAME_CONFIG, "your-key-name");
            map.put(MediumMessagingServiceAzureServiceBusDriverConfig.CREDENTIAL_KEY_CONFIG, "your-key");
            map.put(MediumMessagingServiceAzureServiceBusDriverConfig.PROCESSOR_ENABLED_CONFIG, "true");
            map.put(MediumMessagingServiceAzureServiceBusDriverConfig.PROCESSOR_QUEUE_CONFIG, "QUEUE_NAME");
            
            MediumMessagingServiceConfig config = new MediumMessagingServiceConfig(originals);        
            MediumMessagingService instance = new MediumMessagingService(config);

            instance.addMessageListener(TestMessage.class, new MediumMessageListener<TestMessage>() {
                @Override
                public String getName() {
                    return "test-listener";
                }

                @Override
                public void onMessage(TestMessage message) throws MediumMessageListenerException {
                    
                    logger.info("Received message: " + message + ", Metadata:" + gson.toJson(message.getMetadata()));                
                    
                }
            });


            instance.start();

            logger.info("Messaging service started");

            Runtime.getRuntime().addShutdownHook(new Thread() {
                @Override
                public void run() {
                    logger.info("Stopping messaging service");
                    instance.stop();
                }
            });
    
            

        }
    }
