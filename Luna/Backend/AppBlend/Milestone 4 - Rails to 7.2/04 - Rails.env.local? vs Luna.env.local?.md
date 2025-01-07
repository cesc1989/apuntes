# Rails.env.local? vs Luna.env.local?

Relacionado con y dando respuesta a [[Rails.env.local?]]

No hubo ningún problema. Son implementaciones que van por lados distintos.

```bash
Loading development environment (Rails 7.1.4)
irb(main):001> Rails.env.local?
=> true
irb(main):002> Luna.env.local?
=> true
irb(main):003> Luna.env.live?
=> false
irb(main):004> Rails.env.production?
=> false
irb(main):005> Luna.env.omega?
=> false
```

Esto anterior lo hice en Cash Flow. Archivos a tener en cuenta para replicar el setup:

- `config/boot.rb`
- `config/luna.rb`
- `lib/lunacare.rb`
- `lib/lunacare/`
	- Esto es una carpeta

