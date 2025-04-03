# Otros Acortadores

Bien sean servicios activos o proyectos open source.

## Internacionales

- TinyZap -> [https://tinyzap.com/](https://tinyzap.com/)
- Bitly -> [https://bitly.com/](https://bitly.com/)
- TinyURL -> [https://tinyurl.com/](https://tinyurl.com/)
- Dub -> [https://dub.co](https://dub.co)
- Rebrandly -> [https://www.rebrandly.com/](https://www.rebrandly.com/)

## De Latam

- Lowerhot -> [https://lowerhot.com/](https://lowerhot.com/)
- Acortar link -> [https://acortar.link/](https://acortar.link/)
- Ito -> [http://ito.mx/](http://ito.mx/)
- N9 -> [https://n9.cl/](https://n9.cl/)
- AcortaYa -> [https://acortaya.com/](https://acortaya.com/)

## Open Source

### Dub
Repo -> https://github.com/dubinc/dub

Hecho en NextJS y TypeScript.

El esquema de Dub es más completo. Aquí se pueden ver todos los esquemas -> https://github.com/dubinc/dub/tree/main/packages/prisma/schema

### YLL

Repo -> https://github.com/davidesantangelo/yll

Hecho en Ruby on Rails.

Este es el esquema. Solo tiene una tabla.

```ruby
  create_table "links", force: :cascade do |t|
    t.string "url"
    t.string "password_digest"
    t.datetime "expires_at"
    t.string "code"
    t.integer "clicks", default: 0
    t.datetime "created_at", null: false
    t.datetime "updated_at", null: false
    t.index ["clicks"], name: "index_links_on_clicks"
    t.index ["code"], name: "index_links_on_code", unique: true
    t.index ["url"], name: "index_links_on_url"
  end
```

El campo `password_digest` es para generar enlaces protegidos por contraseña.

### RailsUrlShortener

Repo -> https://github.com/a-chacon/rails-url-shortener/

Hecho en Ruby on Rails. Funciona como engine.

Este proyecto define varias tablas:
```ruby
create_table :rails_url_shortener_urls do |t|
	# optional if you can link it to a user or other model
	t.references :owner, polymorphic: true, null: true
	t.text :url, null: false, length: 2048 # the real url
	t.string :key, limit: 10, null: false # the unique key
	t.string :category # category for short url
	t.datetime :expires_at # valid until date for expirable urls
	t.timestamps
end

create_table :rails_url_shortener_visits do |t|
	t.belongs_to    :url
	t.string        :ip # client ip
	t.string        :browser # browser from user_agent
	t.string        :browser_version # browser version from user_agent
	t.string        :platform # platform from user_agent
	t.string        :platform_version # platform version from user_agent
	t.boolean       :bot # if the request was a bot or not
	t.string        :user_agent
	
	# variable where we save all data that can be catch from the request
	t.text          :meta
	t.timestamps
end

create_table :rails_url_shortener_ipgeos do |t|
	t.string  :ip
	t.string  :country
	t.string  :country_code
	t.string  :region
	t.string  :region_name
	t.string  :city
	t.string  :lat
	t.string  :lon
	t.string  :timezone
	t.string  :isp
	t.string  :org
	t.string  :as
	t.boolean :mobile
	t.boolean :proxy
	t.boolean :hosting
	t.timestamps
end
```