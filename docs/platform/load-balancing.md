## Spring Cloud LoadBalancer

Spring Cloud LoadBalancer is a generic abstraction over load balancing algorithms that you can use with service discovery clients like Eureka, Consul, and Zookeeper. It provides a round-robin load balancing implementation by default, but you can also implement your own custom load balancing algorithms.

Key components of Spring Cloud LoadBalancer include:

- **Dependency**: To use Spring Cloud LoadBalancer, you need to include the `spring-cloud-starter-loadbalancer` dependency in your project.

- **Configuration**: By default, Spring Cloud LoadBalancer uses a simple round-robin strategy for load balancing. If you want to customize this, you can create a bean of type `ServiceInstanceListSupplier` that returns a custom list of instances for load balancing.

- **Usage**: You can use the `@LoadBalanced` annotation on a `RestTemplate` or `WebClient.Builder` bean to integrate it with Spring Cloud LoadBalancer. When you make a request through this client, it will automatically be load balanced.
