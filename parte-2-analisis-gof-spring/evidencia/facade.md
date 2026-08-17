# Patrón 2 — Facade

**Clase:** `org.springframework.jdbc.core.JdbcTemplate`

**Módulo:** spring-jdbc

**Archivo:** `spring-jdbc/src/main/java/org/springframework/jdbc/core/JdbcTemplate.java`

**Commit verificado:** `9d7702eb70767e4dd3e57b71dfb2276dc2e69293` (rama main, 1825 líneas totales)

**Método:** `execute(StatementCallback<T> action)`

**Ubicación exacta:** líneas 437-440, commit `9d7702eb70767e4dd3e57b71dfb2276dc2e69293` (rama main de spring-projects/spring-framework)

**Fuente:** https://github.com/spring-projects/spring-framework/blob/main/spring-jdbc/src/main/java/org/springframework/jdbc/core/JdbcTemplate.java#L437C2-L440C3

## Evidencia de código

```java
public <T> T execute(StatementCallback<T> action) throws DataAccessException {
    return execute(action, true);
}

// Método privado que hace el trabajo pesado (ocultado al desarrollador):
private <T> T execute(StatementCallback<T> action, boolean closeResources) {
    Connection con = DataSourceUtils.getConnection(obtainDataSource());
    Statement stmt = null;
    try {
        stmt = con.createStatement();
        applyStatementSettings(stmt);
        T result = action.doInStatement(stmt);
        handleWarnings(stmt);
        return result;
    }
    catch (SQLException ex) {
        // traduce SQLException -> DataAccessException (no verificada)
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