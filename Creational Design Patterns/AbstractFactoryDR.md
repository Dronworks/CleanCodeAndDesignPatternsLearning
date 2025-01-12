
### Abstract Factory Design Pattern
**Why?**
#### Move object creation logic to a separate class
- Abstract factory is used when we have two or more objects which work together forming a kit or set and there can be multiple sets or kits that can be created by client code.
- So we separate client code from concrete objects forming such a set and also from the code which creates these sets.

**What is a simple factory?**

For example a game with different ages and in medival age: 
* a naval unit - is a viking sheep
* a soldier unit - is a viking

Now you progress to industrial age:
* a naval unit - is a stem ship
* a soldier unit - is a rifleman

Your class will ask for a naval unit, and the Factory will decide what age is it and build the unit accordingly, and return the class to you.

![UML](/Files/AbstractFactoryDP.png)
![UML2](/Files/AbstractFactoryDP2.png)

**Implement a factory**
- Create AbstractProductA and 2 implementations ProductAOpt1 and ProductAOpt2
- Create AbstractProductB and 2 implementations ProductBOpt1 and ProductBOpt2
- Create Abstract factory that defines abstract methods for creating the products
- Provide concrete implementation of factory for each set of products

    **NOTE:** abstract factory uses factory method pattern (it is like an object with multiple factory methods)

**Down side of this Pattern**
- We will have to add more classes in order to create new types of interface object, hence more unit tests

**Examples:**
- Cloud example:
    - A support of the code both for Google cloud and AWS cloud:
        - we need Instance (EC2/GCEI)
        - we need Storage (S3/GCS)
        - a factory to create an instance and a storage
    ![UML](/Files/AbstractFactoryExample.png)
    - Instance Interface
    ```
    //Represents an abstract product
    public interface Instance {
        enum Capacity{micro, small, large}

        void start();

        void attachStorage(Storage storage);

        void stop();

    }
    ```
    - This is a EC2 instance
    ```
    public class Ec2Instance implements Instance {

        public Ec2Instance(Capacity capacity) {
            //Map capacity to ec2 instance types. Use aws API to provision
            System.out.println("Created Ec2Instance");
        }

        @Override
        public void start() {
            System.out.println("Ec2Instance started");
        }
        ...
    }
    ```
    - This is a GoogleComputeEngine instance
    ```
    public class GoogleComputeEngineInstance implements Instance {

        public GoogleComputeEngineInstance(Capacity capacity) {
            //Map capacity to GCP compute instance types. Use GCP API to provision
            System.out.println("Created Google Compute Engine instance");
        }

        @Override
        public void start() {
            System.out.println("Compute engine instance started");
        }
        ...
    }
    ```
    - Storage Interface
    ```
    public interface Storage {
        String getId();
    }
    ```
    - This is a S3 implementation
    ```
    public class S3Storage implements Storage {

        public S3Storage(int capacityInMib) {
            //Use aws s3 api
            System.out.println("Allocated "+capacityInMib+" on S3");
        }

        @Override
        public String getId() {
            return "S31";
        }
        ...
    }
    ```
    - This is a GoogleCloudStorage implementation
    ```
    public class GoogleCloudStorage implements Storage {

        public GoogleCloudStorage(int capacityInMib) {
            //Use gcp api
            System.out.println("Allocated "+capacityInMib+" on Google Cloud Storage");
        }

        @Override
        public String getId() {
            return "gcpcs1";
        }
        ...
    }
    ```
    - Resource factory interface
    ```
    public interface ResourceFactory {

        Instance createInstance(Instance.Capacity capacity);

        Storage createStorage(int capMib);

    }
    ```
    - This is the AWS resource factory implementation
    ```
    public class AwsResourceFactory implements ResourceFactory {

        @Override
        public Instance createInstance(Capacity capacity) {
            return new Ec2Instance(capacity);
        }

        @Override
        public Storage createStorage(int capMib) {
            return new S3Storage(capMib);
        }

    }
    ```
    - This is the Google resource factory implementation
    ```
    public class GoogleResourceFactory implements ResourceFactory {

        @Override
        public Instance createInstance(Capacity capacity) {
            return new GoogleComputeEngineInstance(capacity);

        }

        @Override
        public Storage createStorage(int capMib) {
            return new GoogleCloudStorage(capMib);
        }

    }
    ```
    - **Usage** Client class
    ```
    public class Client {

        private ResourceFactory factory;

        public Client(ResourceFactory factory) {
            this. factory = factory;
        }

        public Instance createServer(Instance.Capacity cap, int storageMib) {
            Instance instance = factory.createInstance(cap);
            Storage storage = factory.createStorage(storageMib);
            instance.attachStorage(storage);
            return instance;
        }

        public static void main(String[] args) {
            Client aws = new Client(new AwsResourceFactory());
            Instance il = aws.createServer(Capacity.micro, 20480);
            il.start();
            i1.stop();

            System.out.println("****");
            Client gcp = new Client(new GoogleResourceFactory());
            i1 = gpc.createServer(Capacity.micro, 20480);
            i1.start();
            i1.stop();
        }
    }
    ```