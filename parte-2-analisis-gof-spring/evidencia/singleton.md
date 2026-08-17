# Patrón 1 — Singleton

**Clase:** `org.springframework.beans.factory.support.DefaultSingletonBeanRegistry`

**Módulo:** spring-beans

**Archivo:** `spring-beans/src/main/java/org/springframework/beans/factory/support/DefaultSingletonBeanRegistry.java`

**Método:** `getSingleton(String beanName, boolean allowEarlyReference)`

**Ubicación exacta:** líneas 180-203, commit `9d768a8` (rama main de spring-projects/spring-framework)

**Fuente:** https://github.com/spring-projects/spring-framework/blob/9d768a8/spring-beans/src/main/java/org/springframework/beans/factory/support/DefaultSingletonBeanRegistry.java#L180-L203

## Evidencia de código

```java
protected Object getSingleton(String beanName, boolean allowEarlyReference) {
    // Verificación rápida sin bloqueo completo del registro
    Object singletonObject = this.singletonObjects.get(beanName);
    if (singletonObject == null && isSingletonCurrentlyInCreation(beanName)) {
        singletonObject = this.earlySingletonObjects.get(beanName);
        if (singletonObject == null && allowEarlyReference) {
            // resuelve referencia temprana para dependencias circulares
        }
    }
    return singletonObject;
}
```