# Análisis de Patrones GoF en Spring Framework

**Nombre:** Maria Camila Meza Palacio
**Código:** 02240131006
**Curso:** Patrones de Diseño de Software
**Unidad:** 1
**Fecha:** 17 de agosto de 2026

---

## Introducción

El presente documento tiene como objetivo analizar la aplicación real de patrones de
diseño del catálogo Gang of Four (GoF) dentro del código fuente de Spring Framework,
uno de los frameworks de desarrollo Java más utilizados a nivel empresarial. El análisis
busca conectar la teoría de patrones creacionales, estructurales y de comportamiento
con implementaciones concretas y verificables en un proyecto de software maduro, así
como identificar la relación de cada patrón con los principios SOLID. Como caso de
estudio se toma Spring Boot, dado que su núcleo (el contenedor de Inversión de Control,
el acceso a datos y el sistema de eventos) constituye un ejemplo representativo de cómo
los patrones GoF resuelven problemas de diseño reales a gran escala.

## Análisis de Patrón 1 — Singleton

El patrón Singleton pertenece a la categoría creacional del catálogo GoF y tiene como
propósito garantizar que una clase tenga una única instancia en un contexto determinado,
proporcionando un punto de acceso global controlado a dicha instancia. En Spring
Framework este patrón aparece implementado en la clase
`org.springframework.beans.factory.support.DefaultSingletonBeanRegistry`, ubicada en
el módulo `spring-beans`, la cual constituye el registro central de beans de alcance
singleton dentro del contenedor de IoC.

El problema que resuelve en este contexto es evitar que el contenedor de Spring cree una
nueva instancia de un bean cada vez que este es solicitado por otro componente, lo cual
sería costoso en memoria y podría generar inconsistencias de estado si múltiples partes
de la aplicación esperan compartir la misma instancia (por ejemplo, un `DataSource` o un
`Service` sin estado). En lugar de que cada clase implemente manualmente su propia
lógica de instancia única —como en el patrón Singleton clásico con constructor privado y
método estático `getInstance()`— Spring centraliza esta responsabilidad en el contenedor,
de modo que ningún bean necesita saber que está siendo tratado como singleton.

Como evidencia de código, el método `getSingleton(String beanName, boolean
allowEarlyReference)`, ubicado en las líneas 180 a 203 del archivo
`DefaultSingletonBeanRegistry.java` (commit 9d768a8), muestra el mecanismo interno de
caché de instancias:

```java
protected Object getSingleton(String beanName, boolean allowEarlyReference) {
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

Este fragmento evidencia que el registro mantiene un mapa (`singletonObjects`) que actúa
como caché: si el bean ya fue creado, se retorna la instancia existente en lugar de
construir una nueva, que es precisamente la esencia del patrón Singleton.

En cuanto a los principios SOLID, este patrón refuerza principalmente el **Principio de
Responsabilidad Única (SRP)**, ya que traslada la gestión del ciclo de vida y la unicidad de
instancias fuera de las clases de negocio hacia una clase especializada del framework.
Ninguna clase de servicio necesita implementar lógica de instanciación única propia; su
única responsabilidad sigue siendo su lógica de dominio. Si Spring no aplicara este patrón,
cada desarrollador tendría que implementar manualmente el control de instancia única en
cada bean que debiera compartirse, duplicando código y aumentando el riesgo de errores
de concurrencia.

## Análisis de Patrón 2 — Facade

El patrón Facade pertenece a la categoría estructural del catálogo GoF y su propósito es
proporcionar una interfaz simplificada y unificada a un subsistema complejo, ocultando los
detalles de su funcionamiento interno. En Spring Framework este patrón se manifiesta en
la clase `org.springframework.jdbc.core.JdbcTemplate`, ubicada en el módulo
`spring-jdbc`, dentro del subsistema de acceso a datos.

El problema que resuelve en este contexto es la enorme cantidad de código repetitivo
(boilerplate) que exige la API estándar de JDBC: abrir una `Connection`, crear un
`Statement`, ejecutar la consulta, capturar la `SQLException` (excepción verificada) y
finalmente cerrar todos los recursos en bloques `finally`. Sin una fachada, cada
desarrollador debería repetir este flujo en cada acceso a la base de datos, lo cual es
propenso a errores como fugas de conexiones no cerradas. `JdbcTemplate` encapsula todo
ese flujo repetitivo detrás de un método simple, dejando al desarrollador únicamente la
responsabilidad de proveer la sentencia SQL e interpretar los resultados.

Como evidencia de código, el método público `execute(StatementCallback<T> action)`
delega en un método privado homónimo que concentra toda la complejidad técnica de
JDBC (archivo `JdbcTemplate.java`, commit `9d7702eb70767e4dd3e57b71dfb2276dc2e69293`):

```java
public <T> T execute(StatementCallback<T> action) throws DataAccessException {
    return execute(action, true);
}

private <T> T execute(StatementCallback<T> action, boolean closeResources) {
    Connection con = DataSourceUtils.getConnection(obtainDataSource());
    Statement stmt = null;
    try {
        stmt = con.createStatement();
        T result = action.doInStatement(stmt);
        return result;
    }
    catch (SQLException ex) {
        throw translateException("StatementCallback", getSql(action), ex);
    }
    finally {
        if (closeResources) {
            JdbcUtils.closeStatement(stmt);
            DataSourceUtils.releaseConnection(con, getDataSource());
        }
    }
}
```

Respecto a los principios SOLID, `JdbcTemplate` refuerza principalmente el **Principio de
Responsabilidad Única (SRP)**, al separar la responsabilidad de "gestión técnica de
recursos JDBC" de la responsabilidad de "lógica de acceso a datos" que le corresponde a
las clases `Repository` de la aplicación; y de forma secundaria el **Principio de Inversión
de Dependencias (DIP)**, dado que el código cliente depende de la abstracción que ofrece
la fachada y no de los detalles concretos del driver JDBC. Sin este patrón, Spring Boot
perdería una de sus mayores ventajas frente a JDBC puro: cada aplicación tendría que
reescribir manualmente el manejo de excepciones y recursos.

## Análisis de Patrón 3 — Observer

El patrón Observer pertenece a la categoría de comportamiento del catálogo GoF y su
propósito es definir una dependencia de uno a muchos entre objetos, de modo que cuando
un objeto (el sujeto) cambia de estado o emite un evento, todos sus dependientes
(observadores) sean notificados automáticamente. En Spring Framework este patrón se
implementa mediante la interfaz `org.springframework.context.ApplicationListener`
(rol de observador) y la clase
`org.springframework.context.event.SimpleApplicationEventMulticaster` (rol de
sujeto/emisor), ambas en el módulo `spring-context`.

El problema que resuelve en este contexto es la comunicación desacoplada entre
componentes del contenedor: cuando ocurre un evento relevante —por ejemplo, el cierre
del contexto de Spring o un evento de negocio personalizado— múltiples partes de la
aplicación pueden necesitar reaccionar sin que el componente que origina el evento
conozca explícitamente quiénes son esos interesados. Sin este patrón, el componente
emisor tendría que mantener referencias directas a cada consumidor y llamarlo
manualmente, generando un acoplamiento fuerte.

Como evidencia de código, el método `multicastEvent`, ubicado en las líneas 135 a 146 del
archivo `SimpleApplicationEventMulticaster.java` (commit f60bec9), muestra cómo se
recorre la lista de observadores registrados y se les notifica el evento:

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

Este fragmento evidencia el mecanismo clásico de Observer: iterar sobre la colección de
observadores (`getApplicationListeners`) e invocar a cada uno (`invokeListener`) cuando
ocurre el evento, sin que el emisor conozca los detalles internos de cada listener.

En cuanto a los principios SOLID, este patrón refuerza principalmente el **Principio
Abierto/Cerrado (OCP)**, puesto que es posible agregar nuevos `ApplicationListener` sin
modificar el código del emisor ni del multicaster; y también el **Principio de Inversión de
Dependencias (DIP)**, ya que tanto el emisor como los observadores dependen de la
abstracción `ApplicationEvent`/`ApplicationListener`, no el uno del otro directamente. Si
Spring Boot no contara con este mecanismo, cada nueva funcionalidad que necesitara
reaccionar a cambios del ciclo de vida de la aplicación obligaría a modificar directamente
las clases núcleo del contenedor.

## Conclusiones

El análisis del código fuente de Spring Framework confirma que los patrones de diseño
GoF no son conceptos exclusivamente académicos, sino soluciones estructurales que
sustentan el funcionamiento de frameworks maduros y ampliamente adoptados en la
industria: Singleton controla el ciclo de vida y la unicidad de los beans dentro del
contenedor de IoC, Facade oculta la complejidad de JDBC detrás de una interfaz simple
en `JdbcTemplate`, y Observer permite la comunicación desacoplada entre componentes
mediante el sistema de eventos de la aplicación. En los tres casos, el uso sistemático de
estos patrones está directamente ligado al cumplimiento de principios SOLID como SRP,
OCP y DIP, lo que demuestra que patrones y principios son herramientas complementarias.
La lección más relevante para el diseño propio es que adoptar un patrón no debe ser un
fin en sí mismo, sino la respuesta a un problema concreto de acoplamiento, duplicación o
rigidez identificado previamente en el diseño.

## Referencias

Gamma, E., Helm, R., Johnson, R., & Vlissides, J. (1994). *Design patterns: Elements of
reusable object-oriented software*. Addison-Wesley.

Spring Framework. (2026). *Data access using JDBC: Using the JDBC core classes to
control basic JDBC processing and error handling*. VMware. https://docs.spring.io/spring-framework/reference/data-access/jdbc/core.html

Spring Projects. (2026). *spring-framework* [Repositorio de código fuente]. GitHub.
https://github.com/spring-projects/spring-framework

Refactoring.Guru. (2026). *Design patterns*. https://refactoring.guru/design-patterns