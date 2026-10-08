CREATE TABLE customers (
    id          BIGINT PRIMARY KEY GENERATED ALWAYS AS IDENTITY,
    name        VARCHAR(200) NOT NULL,
    email       VARCHAR(200),
    phone       VARCHAR(50)
);

CREATE TABLE couriers (
    id          BIGINT PRIMARY KEY GENERATED ALWAYS AS IDENTITY,
    name        VARCHAR(200) NOT NULL,
    active      BOOLEAN NOT NULL DEFAULT TRUE
);

CREATE TABLE orders (
    id              BIGINT PRIMARY KEY GENERATED ALWAYS AS IDENTITY,
    tracking_code   VARCHAR(20) NOT NULL UNIQUE,  -- lo que ve el cliente, no expone el id interno
    customer_id     BIGINT NOT NULL REFERENCES customers(id),
    courier_id      BIGINT REFERENCES couriers(id),
    current_status  VARCHAR(30) NOT NULL,         -- desnormalizado para lecturas rápidas
    address         VARCHAR(500) NOT NULL,
    created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at      TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE TABLE order_events (
    id              BIGINT PRIMARY KEY GENERATED ALWAYS AS IDENTITY,
    order_id        BIGINT NOT NULL REFERENCES orders(id),
    client_event_id UUID NOT NULL,                -- generado en el celular, clave de idempotencia
    status          VARCHAR(30) NOT NULL,
    courier_id      BIGINT REFERENCES couriers(id),
    occurred_at     TIMESTAMPTZ NOT NULL,         -- cuándo pasó (hora del dispositivo)
    received_at     TIMESTAMPTZ NOT NULL DEFAULT now(),  -- cuándo llegó al servidor
    note            VARCHAR(500),
    UNIQUE (client_event_id)
);

CREATE INDEX ix_orders_customer_status ON orders (customer_id, current_status);
CREATE INDEX ix_orders_courier_status  ON orders (courier_id, current_status);
CREATE INDEX ix_events_order_time      ON order_events (order_id, occurred_at);
