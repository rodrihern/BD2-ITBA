
# TP8 Seguridad

## Ejercicio 1

![](attachments/Pasted%20image%2020260909185813.png)

![](attachments/Pasted%20image%2020260909190642.png)

## Ejercicio 2

![](attachments/Pasted%20image%2020260909190605.png)![](attachments/Pasted%20image%2020260909190704.png)

el host no lo voy a incluir, imaginemos un `@'%'` despeus de cada usuario
a.
```sql
GRANT SELECT ON institucion TO U1 WITH GRANT OPTION
GRANT INSERT ON institucion TO U1 WITH GRANT OPTION
GRANT UPDATE ON institucion TO U1 WITH GRANT OPTION
GRANT DELETE ON institucion TO U1 WITH GRANT OPTION
```
b.
```sql
GRANT SELECT ON voluntario TO U2;
```
c.
```sql
GRANT INSERT ON voluntario TO U2 WITH GRANT OPTION;
```
d.
```sql
GRANT INSERT, UPDATE ON tarea TO PUBLIC;
```

no existe `PUBLIC` en mysql

e.
```sql
REVOKE DELETE ON institucion FROM U1;
```

TLT esta guia