## dependances
```
<dependency>
  <groupId>org.projectlombok</groupId>
  <artifactId>lombok</artifactId>
  <optional>true</optional>
</dependency>
```
```
<dependency>
  <groupId>org.springdoc</groupId>
  <artifactId>springdoc-openapi-webflux-ui</artifactId>
  <version>1.4.3</version>
</dependency>
```

## code java classe principale

@Bean
@Lazy(false)
public List<GroupedOpenApi> apis(SwaggerUiConfigParameters swaggerUiConfigParameters, RouteDefinitionLocator locator) {
  List<GroupedOpenApi> groups = new ArrayList<>();
  List<RouteDefinition> definitions = locator.getRouteDefinitions().collectList().block();
  for (RouteDefinition definition : definitions) {
    log.info("Route definition with id {} and uri {}", definition.getId(), definition.getUri().toString());
  }

  definitions.stream().filter(routeDefinition -> routeDefinition.getId().matches(".*-SERVICE")).forEach(routeDefinition -> {
    String name = routeDefinition.getId().replaceAll("ReactiveCompositeDiscoveryClient_", "").toLowerCase();
    log.info("nouveau groupe {}", name);
    swaggerUiConfigParameters.addGroup(name);
    GroupedOpenApi.builder().pathsToMatch("/" + name + "/**").group(name).build();
  });

  return groups;
}

@Bean
public RouteLocator routes(
    RouteLocatorBuilder builder) {
  return builder.routes()
      .route("openapi", r -> r.path("/v3/api-docs/**")
          .filters(f ->
              f.rewritePath("/v3/api-docs/(?<path>.*)", "/$\\{path}/v3/api-docs"))
          .uri("http://localhost:8072"))
      .build();
}

## tests

http://localhost:8072/swagger-ui.html
