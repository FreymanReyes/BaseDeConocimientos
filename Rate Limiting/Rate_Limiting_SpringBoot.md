# 🛡️ Guía de Implementación: Rate Limiting en Bank Dugongo
Esta guía detalla cómo proteger tus microservicios (como customer-service) contra ráfagas de peticiones, asegurando que la infraestructura de Bank Dugongo sea resiliente y profesional.

## 1. Configuración de Dependencias (pom.xml)
Para que las anotaciones funcionen, necesitamos el "cerebro" de Resilience4j y el "motor" de Aspectos de Spring (AOP). Sin AOP, la anotación @RateLimiter es un simple comentario.

XML

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-aop</artifactId>
    </dependency>

    <dependency>
        <groupId>io.github.resilience4j</groupId>
        <artifactId>resilience4j-spring-boot3</artifactId>
        <version>2.2.0</version>
    </dependency>

## 2. Definición del Error (Dominio e Infraestructura)
Primero, registramos el error en nuestro enumerado de mensajes globales.

Archivo: common.error.ErrorCode.java

Java

    public enum ErrorCode {
        // ... otros errores
        TOO_MANY_REQUESTS("Rate limit exceeded. Please wait a moment before trying again");

        private final String message;
        ErrorCode(String message) { this.message = message; }
    }

Luego, enseñamos al GlobalExceptionHandler a reconocer la excepción específica que lanza la librería cuando se agotan los permisos.

Archivo: common.exception.GlobalExceptionHandler.java

Java

    import io.github.resilience4j.ratelimiter.RequestNotPermitted;

    // ... dentro de la clase
    @ExceptionHandler({RequestNotPermitted.class})
    public ResponseEntity<ErrorResponse> handleRequestNotPermitted(RequestNotPermitted ex, HttpServletRequest request){
        return buildResponse(
            ErrorCode.TOO_MANY_REQUESTS.name(),
            ErrorCode.TOO_MANY_REQUESTS.getMessage(),
            request,
            HttpStatus.TOO_MANY_REQUESTS // HTTP 429
        );
    }

## 3. Configuración de Perfiles (application.yml)
Es crucial que el bloque resilience4j esté en la raíz del YAML (fuera del bloque spring).

### Perfil de Desarrollo (application-dev.yml)
Configuración estricta para validar que el error se dispara correctamente en tus pruebas.

YAML

    resilience4j:
    ratelimiter:
        instances:
        customer_service_Limit:
            limitForPeriod: 3        # Permite solo 3 intentos
            limitRefreshPeriod: 10s   # Se recargan cada 10 segundos
            timeoutDuration: 0s      # No espera; falla de inmediato si no hay cupo

### Perfil de Producción (application-prod.yml)
Configuración real para la instancia EC2 de AWS.

YAML

    resilience4j:
    ratelimiter:
        instances:
        customer_service_Limit:
            limitForPeriod: 50       # 50 peticiones
            limitRefreshPeriod: 1m   # Por cada minuto
            timeoutDuration: 0s

## 4. Implementación en el Controlador (Web Layer)
Simplemente "etiquetamos" el método que queremos proteger. El nombre en @RateLimiter debe ser idéntico al definido en el YAML.

Archivo: api.v1.customer.CustomerController.java

Java

    @PostMapping
    @RateLimiter(name = "customer_service_Limit")
    public ResponseEntity<CustomerResponseDTO> create(@Valid @RequestBody CustomerRequestDTO request) {
        return ResponseEntity.status(HttpStatus.CREATED).body(service.create(request));
    }

## 5. ¿Cómo funciona por dentro?
Cuando un usuario envía una petición:

AOP Interceptor: Antes de entrar al método create, Spring pregunta a Resilience4j: "¿Hay permisos disponibles para 'customer_service_Limit'?".

Si hay cupo: Se descuenta 1 del limitForPeriod y la ejecución continúa normal.

Si NO hay cupo: Se lanza la excepción RequestNotPermitted.

Catch: Tu GlobalExceptionHandler captura la excepción y devuelve el JSON con el traceId y el mensaje de error de Bank Dugongo.

## 🚀 Prueba de Fuego (Checklist)
Asegúrate de que en application.yml tengas active: dev.

Haz 4 peticiones seguidas en Postman (en menos de 10 segundos).

Resultado esperado: Las primeras 3 funcionan (201 Created), la 4ta devuelve un 429 Too Many Requests.