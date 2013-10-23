---
layout: post
title: "Rails表的关联"
date: 2013-10-22 21:47
comments: true
categories: [Rails]
---

一段时间没有用到表的关联，回想如何使用时，知识却模糊不清，所以将它记下以便以后查看。
Rails中表的关联主要有三种：单表继承,多态关联,自引用。

个人觉得：

- **单表继承**是通过增加**冗余字段**来**减少表的数量**。

- **多态关联**是通过建立**虚拟表**来**减少关联数**（belongs\_to）和**字段**（外键），方便于以后又有表要关联。

- **自引用**也是利用**冗余**而共用**一张表**。

共同特点：
> 都需要增加type字段，即增加冗余字段。

差别：
> 多态可以让子类有自己的行为特征（通过type字段呈现"多重形态"）。
> 单表继承必须拥有共同属性。
> 自引用必须牺牲一些字段（不需用到的字段的值为null）。

--------------------------------------------------------------

###详述

####单表继承：

一个继承体系**所有类映射到同一张数据库表**，这张表包含**所有类拥有的属性**。记住：所有属性哦，他们可以为null值（:null=>true，即null是可有可无的）。
它通过一个**附加字段type来确认当前记录的对象**属于什么**类型**----ActiveRecord约定。

顾名思义，单表就是一张表，那么怎么呈现多种表的形态呢。当其他表引用它时，可以给予其他名字，但是要声明它实际是那张表(belongs\_to :manager, :class\_name => "Person")，举例：

person.rb:
	Class Person < ActiveRecord::Base
			belongs_to XXX
	end

manager.rb:(继承Person类)
	Class Manager < Person
	end
 

rails console:
	XXX.first.manager
mysql:
	select people.* from people where people.type in ("Manager") and people.id=1

------------------------------------------------------------------

####多态关联：

通过建立虚拟表，通过虚拟表（先通过**类型找到关联表**，再通过**外键找到对应的记录**）访问。

**举例：**

假设我們已经有了Article与Photo这两个Model，然后我们希望这两个Model都可以被留言。不用多态关联的话，你得分別建立ArticleComment和PhotoComment的model。或者一个comment中要有两个外键：article\_id和photo\_id，虽然用多态也是两个字段，但当表多了就可以看出多态的好处。

现有个场景，有两个model：person和album，需要添加一个图片来做为其头像/封面。添加一个image model，按照以往需要对这几个model做以下关联设置：

没用多态关联前：不复杂，但是麻烦，如果**以后加个book model之类**的，也需要有个图做封面的，那又要**改image model里的关联和migration**了，一点都不DRY。

    class Person < ActiveRecord::Base
      has_one :image, :dependent => :destroy
    end

	  class Album < ActiveRecord::Base
			 has_one :image, :dependent => :destroy
	  end

	  class Image < ActiveRecord::Base
	     belongs_to :person
			 belongs_to :album
    end

相应的image的migration要添加上关联字段：

  	t.column :person_id, :integer, :null => false, :default => 0
		t.column :album_id, :integer, :null => false, :default => 0

使用多态关联后：

		class Person < ActiveRecord::Base
			has_one :image, :as => :iconable, :dependent => :destroy
		end
		class Album < ActiveRecord::Base
			has_one :image, :as => :iconable, :dependent => :destroy
		end
		class Image < ActiveRecord::Base
			belongs_to :iconable, :polymorphic => true
		end

Person和Album有了**共同的一个虚拟的名字**叫做iconable,image表就可以直接用外键iconable\_id关联它,

**使用方法：**

添加：
		@person = Person.new(params[:person])
		@person.build_image(params[:image])
		@person.save

读取：
		@person.image

image 的属主：
		@person.iconable

-------------------------------------------------------------------

####自引用：

自引用就是**一条记录可能引用同一张表的另一条记录**：公司员工都有主管，主管也是员工。使用:class\_name 和foregin\_key加上has\_many/has\_one和belongs\_to.

	class Employee < ActiveRecord::Base
		belongs_to :manager,
		  :class_name => "Employee",
		  :foregin_key => "manager_id"
		belongs_to :mentor,
			:class_name => "Employee",
			:foregin_key => "mentor_id"
		has_many ：mentored_employees,
	 	  :class_name => "Employee",
		  :foreign_key => "mentor_id"
		has_many ：managed_employees,
		  :class_name => "Employee",
		  :foreign_key => "manager_id"
	end
