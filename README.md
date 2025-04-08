# Spring-Questions

## Spring-Configuration

**Spring Questions**
**Spring Configuration**

**Questions:**

* ¿Cuál es la diferencia entre las anotaciones `@Configuration`, `@Component` y `@Service`?
* ¿Cómo podemos personalizar el proceso de escaneo de componentes?
* ¿Qué valor tendrá una propiedad si está definida en dos perfiles diferentes, ambos activos?
* ¿Por qué usaríamos fábricas de beans en lugar de beans regulares?
* ¿Cómo podemos sobrescribir cualquier propiedad definida en archivos `.properties`?
* ¿Se llama a `@PostConstruct` para un bean de alcance prototipo?

### ¿Cuál es la diferencia entre las anotaciones `@Configuration`, `@Component` y `@Service`?

1.  `@Configuration`: Se usa para marcar una clase como fuente de definiciones de beans en Spring, permitiendo declarar beans mediante métodos `@Bean`. Típicamente utilizado para configuración basada en Java.
    ```java
    @Configuration
    public class AppConfig {
        @Bean
        public MyService myService() {
            return new MyService();
        }
    }
    ```

2.  `@Component`: Anotación genérica que marca una clase como un bean gestionado por Spring, detectado automáticamente durante el escaneo de componentes.
    ```java
    @Component
    public class MyComponent {
        // lógica del componente
    }
    ```

3.  `@Service`: Especialización de `@Component` para marcar clases de servicio en la capa de negocio, indicando que realizan tareas relacionadas con el negocio. Su comportamiento es similar a `@Component`.
    ```java
    @Service
    public class MyService {
        public void performAction() {
            // lógica de negocio
        }
    }
    ```

**Diferencias Clave:** `@Configuration` para definición programática de beans, `@Component` para registro genérico de beans, y `@Service` como especialización semántica de `@Component` para la capa de servicio.

### ¿Cómo podemos personalizar el proceso de escaneo de componentes?

1.  **`@ComponentScan(basePackages = "...")`**: Limita el escaneo a los paquetes especificados.
    ```java
    @Configuration
    @ComponentScan(basePackages = "com.miempresa.miproyecto.servicios")
    public class AppConfig {
    }
    ```

2.  **`@ComponentScan(basePackageClasses = MiClase.class)`**: Escanea los paquetes que contienen las clases especificadas.
    ```java
    @Configuration
    @ComponentScan(basePackageClasses = MiClaseDeServicio.class)
    public class AppConfig {
    }
    ```

3.  **`@ComponentScan(excludeFilters = @ComponentScan.Filter(...))`**: Excluye clases que coinciden con los filtros definidos (por ejemplo, por anotación).
    ```java
    @Configuration
    @ComponentScan(
        basePackages = "com.miempresa.miproyecto",
        excludeFilters = @ComponentScan.Filter(type = FilterType.ANNOTATION, value = NoEscanear.class)
    )
    public class AppConfig {
    }
    ```

4.  **`@ComponentScan(includeFilters = @ComponentScan.Filter(...))`**: Incluye solo clases que coinciden con los filtros definidos.
    ```java
    @Configuration
    @ComponentScan(
        basePackages = "com.miempresa.miproyecto",
        includeFilters = @ComponentScan.Filter(type = FilterType.ANNOTATION, value = MiAnotacion.class)
    )
    public class AppConfig {
    }
    ```

5.  **`@Profile("...")`**: Activa el escaneo solo si el perfil especificado está activo.
    ```java
    @Configuration
    @ComponentScan(basePackages = "com.miempresa.miproyecto")
    @Profile("produccion")
    public class AppConfig {
    }
    ```

6.  **`@Import(MiConfiguracionAdicional.class)`**: Incluye clases de configuración adicionales sin que sean parte del escaneo automático.
    ```java
    @Configuration
    @Import(MiConfiguracionAdicional.class)
    public class AppConfig {
    }
    ```

### ¿Qué valor tendrá una propiedad si está definida en dos perfiles diferentes, ambos activos?

El valor de la propiedad será determinado por el perfil que tenga la **última definición** o **mayor prioridad** según el orden de activación de los perfiles. La última definición en los perfiles activos que se cargan tendrá precedencia.

### ¿Por qué usaríamos fábricas de beans en lugar de beans regulares?

Los Factory Beans se utilizan para tener **más control sobre el proceso de creación del bean** en los siguientes casos:

1.  **Lógica Compleja de Creación:** Cuando la inicialización del bean requiere pasos complejos o lógica condicional.
2.  **Dependencias Externas:** Cuando la creación del objeto depende de factores externos o servicios.
3.  **Inicialización Perezosa:** Para inicializar el bean solo cuando sea necesario o bajo ciertas condiciones.
4.  **Reutilización y Separación de Responsabilidades:** Para separar la lógica de creación de la lógica de negocio, facilitando la reutilización.
5.  **Creación de Beans de Tipos Variables:** Cuando el tipo del bean a crear depende de condiciones dinámicas.

### ¿Cómo podemos sobrescribir cualquier propiedad definida en archivos `.properties`?

1.  **Argumentos de Línea de Comandos:** Pasar valores como `--propiedad=nuevoValor` al ejecutar la aplicación.
2.  **Perfiles:** Definir archivos de propiedades específicos para perfiles (e.g., `application-dev.properties`) y activar el perfil deseado (`spring.profiles.active=dev`).
3.  **Variables de Entorno:** Establecer variables de entorno (e.g., `MI_PROPIEDAD=nuevoValor`), que Spring Boot mapea automáticamente a las propiedades.
4.  **Anotaciones `@Value` o `@ConfigurationProperties`:** Los valores inyectados por estas anotaciones pueden ser sobrescritos por los métodos anteriores.
5.  **`application.yml`:** Las mismas técnicas de sobrescritura aplican para archivos en formato YAML.

### ¿Se llama a `@PostConstruct` para un bean de alcance prototipo?

No, el método `@PostConstruct` **no se llama automáticamente** para beans de alcance `prototype`. Spring crea nuevas instancias de beans prototype cada vez que se solicitan, pero no gestiona su ciclo de vida más allá de la creación. Por lo tanto, no se invoca ningún método de inicialización o destrucción gestionado por Spring para estos beans. La responsabilidad de cualquier inicialización o limpieza recae en el código cliente que solicita y utiliza el bean prototype.
