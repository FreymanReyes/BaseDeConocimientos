# 🚀 Guía de Implementación: Idempotencia con Redis en Spring Boot 3.x

Este documento detalla la arquitectura y configuración necesaria para evitar transacciones duplicadas en el microservicio de Bank Dugongo utilizando Redis como capa de caché de corta duración.

## 🛠️ 1. Infraestructura: Configuración con Docker

Para levantar una instancia limpia de Redis, utilizamos la imagen ligera de Alpine:PowerShell

# Comando para levantar el contenedor

docker run --name redis-dugongo -p 6379:6379 -d redis:alpine

### Comandos Útiles de Administración (Redis-CLI)

Acceso al contenedor: **docker exec -it redis-dugongo redis-cli**


#### Acción: 
🔍 Ver todas las llaves,
#### Comando:
keys *
#### Respuesta Esperada:
Lista de llaves o (empty array)


#### Acción: 
🗑️ Borrar una llave
#### Comando:
"DEL ""IdempotencyKey:uuid"""
#### Respuesta Esperada:
(integer) 1 (Éxito) o 0 (No existe)

#### Acción: 
🧹 Borrar TODO
#### Comando:
FLUSHALL
#### Respuesta Esperada:
OK


#### Acción: 
📄 Ver contenido
#### Comando:
"HGETALL ""IdempotencyKey:uuid"""
#### Respuesta Esperada:
Lista de campos y valores


#### Acción: 
⚡ Monitorear en vivo
#### Comando:
MONITOR
#### Respuesta Esperada:
Flujo de comandos en tiempo real


## 📦 2. Dependencias (pom.xml)

Necesitamos el starter de Spring Data Redis para habilitar los Repositorios y la conexión con el servidor.

XML

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-redis</artifactId>
    </dependency>


## 💾 3. Capa de Datos: La Entidad y el Repositorio

### IdempotencyEntry.java (Entidad Redis)

Es fundamental usar @RedisHash para definir el prefijo y el tiempo de vida (TTL) de la llave.

    @Getter
    @Setter
    @Builder
    @NoArgsConstructor
    @AllArgsConstructor
    @RedisHash(value = "IdempotencyKey", timeToLive = 86400) // 24 horas de persistencia
    public class IdempotencyEntry implements Serializable {
        @Id // Importar de org.springframework.data.annotation.Id
        private String key;
        
        private String responseBody;
        private int statusCode;
    }


### IdempotencyRedisRepository.java

Extendemos de CrudRepository para manejar persistencia de forma automática.

Java

@Repository
public interface IdempotencyRedisRepository extends CrudRepository<IdempotencyEntry, String> {
}

## 🧠 4. Flujo Lógico: Interceptor y Advice

### Paso A: El Interceptor (preHandle) - "El Portero"

Misión: Revisar si la llave ya existe antes de que la petición llegue al Controller.



    @Component
    @RequiredArgsConstructor
    public class IdempotencyInterceptor implements HandlerInterceptor {
        private final IdempotencyRedisRepository redisRepository;
    
        @Override
        public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
            if (!"POST".equalsIgnoreCase(request.getMethod())) return true;
    
            String key = request.getHeader("X-Idempotency-Key");
            if (key == null || key.isBlank()) throw new ApiException(ErrorCode.INVALID_HEADER_IDEMPOTENCY);
    
            // BUSQUEDA EN REDIS
            return redisRepository.findById(key).map(entry -> {
                try {
                    // SI EXISTE: Frenamos la ejecución y devolvemos la respuesta guardada
                    response.setStatus(entry.getStatusCode());
                    response.setContentType("application/json");
                    response.getWriter().write(entry.getResponseBody());
                    response.getWriter().flush();
                    return false; 
                } catch (Exception e) {
                    return false;
                }
            }).orElse(true); // SI NO EXISTE: Permitimos el paso al Controller
        }


### Paso B: El Advice (beforeBodyWrite) - "El Notario"

Misión: Capturar la respuesta exitosa del Controller y guardarla en Redis. Se activa gracias a @RestControllerAdvice.


    @RestControllerAdvice 
    @RequiredArgsConstructor
    public class IdempotencyResponseAdvice implements ResponseBodyAdvice<Object> {
        private final IdempotencyRedisRepository redisRepository;
        private final ObjectMapper objectMapper;

    @Override
    public boolean supports(MethodParameter returnType, Class converterType) {
        return true; 
    }

    @Override
    public Object beforeBodyWrite(Object body, MethodParameter returnType, MediaType selectedContentType,
                                  Class selectedConverterType, ServerHttpRequest request, ServerHttpResponse response) {
        
        String key = request.getHeaders().getFirst("X-Idempotency-Key");
        int status = ((ServletServerHttpResponse) response).getServletResponse().getStatus();

        // GUARDADO EN REDIS: Solo si hay llave y la respuesta es exitosa (2xx)
        if (key != null && body != null && status >= 200 && status < 300) {
            try {
                IdempotencyEntry entry = IdempotencyEntry.builder()
                        .key(key)
                        .responseBody(objectMapper.writeValueAsString(body))
                        .statusCode(status)
                        .build();
                redisRepository.save(entry);
            } catch (Exception ignored) {}
        }
        return body;
    }


## ⚙️ 5. Comunicación y Registro (WebConfig)

Para que el Interceptor sea inyectado en el flujo de peticiones de Spring MVC:



    @Configuration
    @RequiredArgsConstructor
    public class WebConfig implements WebMvcConfigurer {
        private final IdempotencyInterceptor idempotencyInterceptor;

        @Override
        public void addInterceptors(InterceptorRegistry registry) {
            registry.addInterceptor(idempotencyInterceptor)
                    .addPathPatterns("/api/v1/transactions/**");
        }


## 🧪 6. Protocolo de Pruebas en 🟠 Postman 

### ❌ Escenario 1: Sin Header

Acción: Enviar POST sin X-Idempotency-Key.
Resultado: 400 Bad Request.

### ✅ Escenario 2: Primera Petición (Creación)

Acción: Enviar POST con X-Idempotency-Key: tx-unique-001.
Flujo: Interceptor (no encuentra) → Controller (procesa) → Advice (guarda en Redis) → Cliente recibe 201 Created.

### 🛡️ Escenario 3: Segunda Petición (Duplicada)

Acción: Enviar mismo POST con misma llave tx-unique-001.
Flujo: Interceptor (encuentra llave en Redis) → Devuelve JSON cacheado inmediatamente → El Controller NUNCA se ejecuta.

## 📚 Glosario de Anotaciones

@Id: Indica a Spring Data Redis que este campo es la clave primaria del Hash.
@RedisHash: Define el "Namespace" en Redis. Evita que llaves de diferentes microservicios se mezclen.
@RestControllerAdvice: Permite interceptar el cuerpo de la respuesta de forma global antes de que se escriba en el stream de salida.
