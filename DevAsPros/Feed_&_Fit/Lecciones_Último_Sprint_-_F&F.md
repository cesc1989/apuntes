# Lecciones Último Sprint - F&F

## `[has_many](https://apidock.com/rails/ActiveRecord/Associations/ClassMethods/has_many)` [sintaxis](https://apidock.com/rails/ActiveRecord/Associations/ClassMethods/has_many) y bloque para extender el macro


    has_many(name, scope = nil, **options, &extension) public

El último argumento, `&extension` es un bloque para definir otras formas de llamar al macro:


    has_many :children, :dependent => :destroy do
      def at(time)
        proxy_owner.children.find_with_deleted :all, :conditions => [
          "created_at <= :time AND (deleted_at > :time OR deleted_at IS NULL)", { :time => time }
        ]        
      end      
    end
    
    Model.children.each # do stuff
    Model.children.at( 1.week.ago ).each # do old stuff

Con un scope para filtrar por defecto la asociación:

    has_many :comments, -> { where(author_id: 1) }
    has_many :employees, -> { joins(:address) }
    has_many :posts, ->(blog) { where("max_post_length > ?", blog.max_post_length) }

