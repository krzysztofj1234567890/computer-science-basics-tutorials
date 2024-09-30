# Event-driven architecture on aws

__design__: event storming -> ( command (play video), query, event (video uploaded) ) -> services -> add communication patterns (queue, pub-sub (kafka, event-bridge), streaming) -> workflows (step functions)

__pros__: decoupled, scalaility, extensibility,  

__issues__ with data in event: need more data (enrich), sensitive data (encrypt), large messages, 

__event schema__

__cross-domain events__ (boundaries)

__standards__: timestamp, corelationId, domain name, 

__discoverability__: schema registry, 