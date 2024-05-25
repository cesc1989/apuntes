# Why I Tend Not To Use Content Negotiation - Apunte
Este ensayo explica las razones por las cuales tener una API de aplicación que retorne solo HTML y una de datos que retorne solo JSON es conveniente.

Comenta que el mecanismo de Content Negotiation de HTTP es bueno pero no conveniente en Hypermedia-Drive Applications.

Menciona el ejemplo de Rails con el método `respond_to` y como se puede devolver HTML y JSON pero eso sería una desventaja en HDA. Lo ideal es tener una API de aplicación (HTML) separada de la API de datos (JSON).


## Ejemplo de Rails
    def index
      @contacts = Contacts.all
    
      respond_to do |format|
        format.html # default rendering logic
        format.json { render json: @contacts }
      end
    end


## So What’s The Problem?
- Data APIs should be versioned and should be very stable within a particular version of the API
- Data APIs typically use some sort of token-based authentication
- Hypermedia APIs should be driven by the needs of the underlying hypermedia application

En resumen:

> Your JSON API needs to be a stable set of endpoint that client code can rely on.
> 
> Your hypermedia API (…) can change dramatically based on the user interface needs of your applications.


## Alternativa a Content Negotiation?

Mantener dos APIs.


> providing different paths (or sub-domains, or whatever) for your JSON API and your hypermedia (HTML) API.


- The JSON API to get all contacts is found at `/api/v1/contacts`
- The Hypermedia API to get all contacts is found at `/contacts`

Esto trae la ventaja de que la API de aplicación puede cambiar a merced de lo que haya que hacer en la UI.

> your hypermedia API (…) can change dramatically as your user interface needs change, with highly tuned database queries, end-points to support special UI needs, etc.


## En Conclusión
> By separating these two concerns, your JSON API can be stable, regular and low-maintenance, and your hypermedia API can be chaotic, specialized and flexible. Each gets its own controller environment to thrive in, without conflicting with one another.

