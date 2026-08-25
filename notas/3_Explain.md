
# Explicando el plan

Se usa para ver que esta haciendo por dentro (**NO** es SQL estandar)


## Explain

```sql
EXPLAIN [ANALYZE] <query>
```

EXPLAIN solo **No** ejecuta la query, estima el costo. Con analyze si. Si quiero asegurarme que no tenga efectos secundarios puedo hacer 

```sql
BEGIN;
EXPLAIN ANALIZE ...;
ROLLBACK;
```

El costo que me da el explain me lo da en una unidad arbitraria (solo sirve para comparar entre queries)


### Ejemplo

![](attachments/Pasted%20image%2020260811180104.png)

## Analyze

```sql
ANALYZE [VERBOSE] [table[(column [,...])]]
```

Se usa para ver estadisticas

### Ejemplo

![](attachments/Pasted%20image%2020260811180718.png)

## Formula para estimar el costo

$$
pages \times seq\_page\_cost + tuples\_to\_return \times cpu\_tuple\_cost
$$
Las estadisticas se actualizan cuando se corre un `ANALYZE`

## Conclusion

Esta bueno para jugar con queries, con indices, etc.