# GraphQL Ruby - En detalle

# Configuración

Gemas

- graphql-ruby → https://github.com/rmosolgo/graphql-ruby
- graphiql-rails → https://github.com/rmosolgo/graphiql-rails


# Campos, Tipos, Resolvers y Queries

Siempre se parte de un Tipo. Para Leisure Shelf Playground quise crear el tipo Activity. Lo hice así:

    rails g graphql:object ActivityType id:ID! name:String! description:String!


> Nota que la sintaxis del tipo des la de GraphQL y no la de Rails

Ese comandó generó este archivo:

    # app/graphql/types/activity_type.rb
    module Types
      class ActivityType < Types::BaseObject
        field :id, ID, null: false
        field :name, String, null: false
        field :description, String, null: false
      end
    end

Luego, yendo a graphiql (`http://localhost:3001/graphiql`) y para poder hacer una consulta hay que definir un Resolver para este tipo.

En este caso es un resolver de Query entonces se crea en el archivo `query_type.rb`

    # app/graphql/types/query_type.rb
    module Types
      class QueryType < Types::BaseObject
        
        # (existing code)...
    
        # Defino un nuevo field    
        field :all_activities, [ActivityType], null: false, description: 'Todas las actividades disponibles'
    
        # Defino el resolver de ese campo/field
        def all_activities
          Activity.order(created_at: :desc)
        end
      end
    end

Y finalmente lo puedo usar en GraphiQL

    # Query
    {
      allActivities {
        id
        name
        description
      }
    }
    
    # respuesta
    {
      "data": {
        "allActivities": [
          {
            "id": "22",
            "name": "Devs",
            "description": "una serie sobre predecir el futuro"
          },
          # ...
        ]
      }
    }


# Mutaciones - Mutations

Enlaces:

- [Docs](https://graphql.org/learn/queries/#mutations) generales

Si se define esta mutación:

    # app/graphql/mutations/create_activity.rb
    module Mutations
      class CreateActivity < BaseMutation
        # Todo argumento que debe recibir el método resolve
        argument :name, String, required: true
        argument :description, String, required: true
    
        # El tipo a retornar
        type Types::ActivityType
    
        def resolve(name:, description:)
          Activity.create!(
            name: name,
            description: description
          )
        end
      end
    end

Hay que definir el Tipo para poder hacer la query de mutación

    # app/graphql/types/mutation_type.rb
    module Types
      class MutationType < Types::BaseObject
        field :create_activity, mutation: Mutations::CreateActivity
      end
    end

Se usaría así en el explorador:

    mutation {
      createActivity(input: {
          name: "Desde GrapqhiQL",
          description: "Aprendiendo"
        }
      ) {
        id
        name
        description
      }
    }


# Pruebas con RSpec

Así escribí una prueba según encontré en [este tutorial](https://dev.to/phawk/screencast-testing-graphql-with-rails-and-rspec-303m):

    # spec/graphql/queries/query_activities_spec.rb
    
    require 'rails_helper'
    
    RSpec.describe 'Query Activities' do
      it 'returns existing activities' do
        create_list(:activity, 3)
    
        query = <<~GQL
          query {
            allActivities {
              id
              name
              description
            }
          }
        GQL
    
        result = LeisureShelfPlaygroundSchema.execute(query)
    
        expect(result).not_to be_empty
        expect(result\['data'\]['allActivities'].first).to include('id', 'name', 'description')
    
        first_activity = result.dig('data', 'allActivities').first
    
        expect(first_activity.dig('name')).to eq('Hola Mundo')
      end
    end
    

