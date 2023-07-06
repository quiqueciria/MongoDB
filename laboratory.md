# Laboratorio MongoDB

**Pregunta**. Si montaras un sitio real, ¿Qué posibles problemas pontenciales les ves a como está almacenada la información?

```md
Indica aquí que problemas ves
```

## Obligatorio

Esta es la parte mínima que tendrás que entregar para superar este laboratorio.

### Consultas

- Saca en una consulta cuantos alojamientos hay en España.

```js
db.listingsAndReviews.countDocuments({
  "address.country": {
    $eq: "Spain",
  },
});
```

- Lista los 10 primeros:
  - Ordenados por precio de forma ascendente.
  - Sólo muestra: nombre, precio, camas y la localidad (`address.market`).

```js
db.listingsAndReviews
  .find(
    {
      $and: [
        {
          "address.country": {
            $eq: "Spain",
          },
        },
        {
          price: {
            $gt: 100.0,
          },
        },
      ],
    },
    {
      _id: 1,
      name: 1,
      price: 1,
      beds: 1,
      "address.market": 1,
    }
  )
  .sort({
    price: 1,
  })
  .limit(10);
```

### Filtrando

- Queremos viajar cómodos, somos 4 personas y queremos:
  - 4 camas.
  - Dos cuartos de baño o más.
  - Sólo muestra: nombre, precio, camas y baños.

```js
db.listingsAndReviews.find(
  {
    beds: {
      $eq: 4,
    },
    bathrooms: {
      $gte: 2,
    },
  },
  {
    _id: 0,
    name: 1,
    price: 1,
    beds: 1,
    bathrooms: 1,
  }
);
```

- Aunque estamos de viaje no queremos estar desconectados, así que necesitamos que el alojamiento también tenga conexión wifi. A los requisitos anteriores, hay que añadir que el alojamiento tenga wifi.
  - Sólo muestra: nombre, precio, camas, baños y servicios (`amenities`).

```js
db.listingsAndReviews.find(
  {
    beds: {
      $eq: 4,
    },
    bathrooms: {
      $gte: 2,
    },
    amenities: {
      $eq: "Wifi",
    },
  },
  {
    _id: 0,
    name: 1,
    price: 1,
    beds: 1,
    bathrooms: 1,
    amenities: 1,
  }
);
```

- Y bueno, un amigo trae a su perro, así que tenemos que buscar alojamientos que permitan mascota (_Pets allowed_).
  - Sólo muestra: nombre, precio, camas, baños y servicios (`amenities`).

```js
db.listingsAndReviews.find(
  {
    beds: {
      $eq: 4,
    },
    bathrooms: {
      $gte: 2,
    },
    amenities: {
      $eq: "Wifi",
      $eq: "Pets allowed",
    },
  },
  {
    _id: 0,
    name: 1,
    price: 1,
    beds: 1,
    bathrooms: 1,
    amenities: 1,
  }
);
```

- Estamos entre ir a Barcelona o a Portugal, los dos destinos nos valen. Pero queremos que el precio nos salga baratito (50 $), y que tenga buen rating de reviews (campo `review_scores.review_scores_rating` igual o superior a 88).
  - Sólo muestra: nombre, precio, camas, baños, rating y localidad.

```js
db.listingsAndReviews.find(
  {
    price: {
      $lte: 50.0,
    },
    security_deposit: {
      $eq: 0,
    },
    "review_scores.review_scores_rating": {
      $gte: 88,
    },
    $or: [
      {
        "address.market": {
          $eq: "Barcelona",
        },
      },
      {
        "address.country": {
          $eq: "Portugal",
        },
      },
    ],
  },
  {
    _id: 0,
    name: 1,
    price: 1,
    beds: 1,
    bathrooms: 1,
    "address.market": 1,
    "review_scores.review_scores_rating": 1,
    security_deposit: 1,
  }
);
```

- También queremos que el huésped sea un superhost (`host.host_is_superhost`) y que no tengamos que pagar depósito de seguridad (`security_deposit`).
  - Sólo muestra: nombre, precio, camas, baños, rating, si el huésped es superhost, depósito de seguridad y localidad.

```js
db.listingsAndReviews.find(
  {
    price: {
      $lte: 50.0,
    },
    security_deposit: {
      $eq: 0,
    },
    "host.host_is_superhost": {
      $eq: true,
    },
    "review_scores.review_scores_rating": {
      $gte: 88,
    },
    $or: [
      {
        "address.market": {
          $eq: "Barcelona",
        },
      },
      {
        "address.country": {
          $eq: "Portugal",
        },
      },
    ],
  },
  {
    _id: 0,
    name: 1,
    price: 1,
    beds: 1,
    bathrooms: 1,
    "address.market": 1,
    "review_scores.review_scores_rating": 1,
    "host.host_is_superhost": 1,
    security_deposit: 1,
  }
);
```

### Agregaciones

- Queremos mostrar los alojamientos que hay en España, con los siguientes campos:
  - Nombre.
  - Localidad (no queremos mostrar un objeto, sólo el string con la localidad).
  - Precio

```js
db.listingsAndReviews.aggregate([
  {
    $match: { "address.country": "Spain" },
  },
  {
    $project: {
      _id: 0,
      name: 1,
      Localidad: "$address.market",
      price: 1,
    },
  },
]);
```

- Queremos saber cuantos alojamientos hay disponibles por pais.

```js
db.listingsAndReviews.aggregate([
  {
    $project: {
      _id: 0,
      "address.country": 1,
    },
  },
  {
    $group: {
      _id: "$address.country",
      Alojamientos: {
        $sum: 1,
      },
    },
  },
]);
```

```
:loudspeaker: ENTREGADA LA PARTE OBLIGATORIA. CONTINUO CON LA OPCIONAL
```

## Opcional

- Queremos saber el precio medio de alquiler de airbnb en España.

```js
// Pega aquí tu consulta
```

- ¿Y si quisieramos hacer como el anterior, pero sacarlo por paises?

```js
// Pega aquí tu consulta
```

- Repite los mismos pasos pero agrupando también por numero de habitaciones.

```js
// Pega aquí tu consulta
```

## Desafio

Queremos mostrar el top 5 de alojamientos más caros en España, con los siguentes campos:

- Nombre.
- Precio.
- Número de habitaciones
- Número de camas
- Número de baños
- Ciudad.
- Servicios, pero en vez de un array, un string con todos los servicios incluidos.

```js
// Pega aquí tu consulta
```
