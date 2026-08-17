# Patrón 3 — Observer

**Clases:**
- `org.springframework.context.ApplicationListener` (observador)
- `org.springframework.context.event.SimpleApplicationEventMulticaster` (sujeto/emisor)

**Módulo:** spring-context

**Archivo:** `spring-context/src/main/java/org/springframework/context/event/SimpleApplicationEventMulticaster.java`

**Método:** `multicastEvent(ApplicationEvent event, ResolvableType eventType)`

**Ubicación exacta:** líneas 135-146, commit `f60bec9` (rama main de spring-projects/spring-framework)

**Fuente:** https://github.com/spring-projects/spring-framework/blob/f60bec9/spring-context/src/main/java/org/springframework/context/event/SimpleApplicationEventMulticaster.java#L135-L146

## Evidencia de código

```java
public void multicastEvent(ApplicationEvent event, ResolvableType eventType) {
    ResolvableType type = (eventType != null ? eventType : ResolvableType.forInstance(event));
    Executor executor = getTaskExecutor();
    for (ApplicationListener<?> listener : getApplicationListeners(event, type)) {
        if (executor != null && listener.supportsAsyncExecution()) {
            executor.execute(() -> invokeListener(listener, event));
        } else {
            invokeListener(listener, event);
        }
    }
}
```