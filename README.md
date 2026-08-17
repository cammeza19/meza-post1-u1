# Post-contenido — Unidad 1: Fundamentos de Patrones de Diseño y Buenas Prácticas

## Descripción
Repositorio del post-contenido de la Unidad 1 de Patrones de Diseño de Software — Sexto Semestre. Contiene dos partes: refactorización SOLID de un God Object (`parte-1-refactorizacion-solid/`) y análisis de patrones GoF en Spring Framework (`parte-2-analisis-gof-spring/`).

## Parte 1 — Refactorización SOLID
Proyecto Maven que refactoriza `OrderProcessor` aplicando SRP, OCP y DIP. Ver `parte-1-refactorizacion-solid/`.

### Análisis de Violaciones SOLID — Parte 1
| Principio | Método/Sección afectada | Descripción de la violación |
|-----------|-------------------------|-----------------------------|
| **SRP** (Single Responsibility Principle) | `calculateTotal`, `applyDiscount`, `saveOrder`, `sendEmail`, `printReport` | La clase `OrderProcessor` asume múltiples responsabilidades no cohesivas: cálculo de impuestos, lógica de descuentos, persistencia en base de datos, envío de notificaciones y generación de reportes en consola. |
| **OCP** (Open/Closed Principle) | `applyDiscount` | El método utiliza condicionales (`if/else`) sobre el tipo de cliente (`VIP`, `REGULAR`). Para agregar un nuevo tipo de descuento o regla comercial, es obligatorio modificar la clase directamente en lugar de extender su comportamiento. |
| **LSP** (Liskov Substitution Principle) | Lógica general de la clase `OrderProcessor` | Al ser una clase concreta acoplada que maneja directamente tipos específicos de clientes mediante condicionales duros, no existe un contrato ni jerarquía de subtipos que garantice el reemplazo seguro de comportamientos sin alterar la ejecución esperada. |
| **ISP** (Interface Segregation Principle) | Interfaz implícita de `OrderProcessor` | La clase expone un conjunto masivo y variado de métodos públicos (`calculateTotal`, `saveOrder`, `sendEmail`, `printReport`), obligando a cualquier cliente que interactúe con ella a depender de métodos que probablemente no necesita ni utiliza. |
| **DIP** (Dependency Inversion Principle) | Toda la clase `OrderProcessor` | La clase depende directamente de implementaciones concretas e internas para la persistencia y la notificación en lugar de depender de abstracciones o interfaces inyectadas desde el exterior. |

## Parte 2 — Análisis de Patrones GoF en Spring
| # | Patrón | Categoría | Clase en Spring |
|---|--------|-----------|-----------------|
| 1 | **Singleton** | Creacional | `org.springframework.beans.factory.support.DefaultSingletonBeanRegistry` |
| 2 | **Facade** | Estructural | `org.springframework.jdbc.core.JdbcTemplate` |
| 3 | **Observer** | Comportamiento | `org.springframework.context.event.SimpleApplicationEventMulticaster` |

Ver `parte-2-analisis-gof-spring/documento-analisis.md`.

## Herramientas utilizadas
- Java 17, Apache Maven, VS Code, Git, GitHub
- Código fuente de Spring Framework (investigación)

## Conclusiones
La refactorización del God Object demostró cómo la aplicación de los principios SOLID transforma código rígido y acoplado en componentes cohesivos, testeables y extensibles mediante patrones como Strategy e Inyección de Dependencias. Asimismo, el análisis del código fuente de Spring Framework evidenció que patrones GoF como Singleton, Facade y Observer son fundamentales en arquitecturas de software maduras para gestionar el ciclo de vida, simplificar subsistemas complejos y habilitar comunicación desacoplada. En conjunto, ambas partes reafirman que la combinación estratégica de principios y patrones de diseño permite construir software escalable y mantenible a largo plazo.