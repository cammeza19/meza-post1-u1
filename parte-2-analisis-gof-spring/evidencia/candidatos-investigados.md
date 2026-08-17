# Candidatos investigados — Fase 1 (Paso 11)

Investigación realizada en el repositorio spring-projects/spring-framework en GitHub,
usando la búsqueda de código del repositorio para localizar las clases candidatas.

| # | Categoría | Patrón | Clase/Interfaz real | ¿Elegido? |
|---|---|---|---|---|
| 1 | Creacional | Singleton | DefaultSingletonBeanRegistry (spring-beans) | Sí |
| 2 | Creacional | Factory Method | AbstractBeanFactory.doGetBean() (spring-beans) | No |
| 3 | Estructural | Facade | JdbcTemplate (spring-jdbc) | Sí |
| 4 | Estructural | Proxy | JdkDynamicAopProxy (spring-aop) | No |
| 5 | Comportamiento | Observer | ApplicationListener / SimpleApplicationEventMulticaster (spring-context) | Sí |

Se eligieron los 3 marcados porque cubren las 3 categorías GoF exigidas por el checkpoint
(al menos uno Creacional, uno Estructural y uno de Comportamiento) y porque tienen
evidencia de código clara y localizable con nombre de paquete completo, guardada en los
archivos singleton.md, facade.md y observer.md de esta misma carpeta.