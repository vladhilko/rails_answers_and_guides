1. How you can create provider app for testing Omniath?
          
          # for twitter
	   	1. https://apps.twitter.com/
	   	# callback (users for devise)
	   	2 http://127.0.0.1:3000/users/auth/twitter/callback
	   
          # for facebook
	   	1. https://developers.facebook.com/
	   	2. create new app, in "проверка приложения" choose available for everybody
	   	3. in setting add new platform (web-site) with such adress (#users need for devise)
	   	http://localhost:3000/users/auth/facebook/callback
	  
	  	# for vk
		1. https://vk.com/dev
		2. choose my_app -> create new app -> web-site
		3. http://localhost:3000/users/auth/vkontakte/callback
		4. Take id - приложения и защищенный ключ
		  # config/initializers/devise.rb
		  config.omniauth :vkontakte, "6062467", "F4ct8LABSyEO3fpPbFWm"
	  
2. Where you have to insert KEY in omniath.rb ?
          
          Rails.application.config.middleware.use OmniAuth::Builder do
            # provider :twitter, ENV['TWITTER_KEY'], ENV['TWITTER_SECRET']
            provider :twitter, '75UOAIDmKrRXvXKBhNvKA', 'GrIaBI0tQy2TtjOtaFL9VxT6s9qq1sV7h9yRaZW4A'
          end
	  
	   
## Omniath with devise

		1. add gems 
			gem 'devise'
			gem 'omniauth'
			gem 'omniauth-twitter'
			gem 'omniauth-facebook'
			gem 'omniauth-vkontakte'
		
		3. generate devise
			rails generate devise:install
			rails generate devise User
			rake db:migrate
		4. add provider and uid column to user
			rails g migration AddColumnsToUsers provider uid
			rake db:migrate
		5. creatate facebook, vk, twitter app ang get secret id
		6. add secret key to config/initialize/devise.rb
			  #ID first, secret second
			  config.omniauth :digitalocean, "db381dc9990be7e3bc42503d0", "5b0824c2722b65d29965f1a1df"
		7. update user class with omniathable, and from_omneath method
			class User < ApplicationRecord
			  devise :database_authenticatable, :registerable,
				 :recoverable, :rememberable, :trackable, :validatable,
				 :omniauthable, :omniauth_providers => [:twitter, :facebook, :vkontakte]
				 
			    def self.from_omniauth(auth)
				    where(provider: auth.provider, uid: auth.uid).first_or_create do |user|
				      user.provider = auth.provider
				      user.uid = auth.uid

				      case user.provider
				      when "facebook"
					user.email = auth.info.email
					user.username = auth.info.name
				      when "vkontakte"
					user.email = auth.info.email.nil? ? "vk#{auth.uid}@tut.by": auth.info.email
					user.username = auth.info.name
				      when "twitter"
					user.email = auth.info.email.nil? ? (auth.info.nickname + '@tut.by') : auth.info.email
					user.username = auth.info.name        
				      end
				      user.password = Devise.friendly_token[0,20]
				    end
			     end
			end
		8. add to routing callback
			# before this action delete devise_for :users, which was generated before
			devise_for :users, :controllers => { :omniauth_callbacks => "callbacks" }
		9. create callback controller  myapp/app/controllers/callbacks_controller.rb
			class CallbacksController < Devise::OmniauthCallbacksController
			    def twitter
				@user = User.from_omniauth(request.env["omniauth.auth"])
				sign_in_and_redirect @user
			    end
			    def facebook
				@user = User.from_omniauth(request.env["omniauth.auth"])
				sign_in_and_redirect @user
			    end
			    def vkontakte
				@user = User.from_omniauth(request.env["omniauth.auth"])
				sign_in_and_redirect @user
			    end
			end
			
## CanCanCan

1. How you can show element only when user has access?
	
		<% if can? :update, @article %>
		  <%= link_to "Edit", edit_article_path(@article) %>
		<% end %>
2. How to translate answer to json?
		
		# add as_json method
		class Ability
		    include CanCan::Ability

		    def initialize(user)
		    end

		  def as_json(options = {})
		    abilities = []
		    rules.each do |rule|
		      abilities << { can: rule.base_behavior, actions: rule.actions.as_json, subjects: rule.subjects.map(&:to_s), conditions: rule.conditions.as_json }
		    end
		    abilities
		  end
		end

## CanCanCan with Devise

1. How to create Devise with CanCanCan?
		
		1. create new app
			rails new devise_cancan
          		rails g scaffold status post:text
		2. add gems
			 gem 'devise'
  			 gem 'cancancan'
		3. generate devise
			rails generate devise:install
			rails generate devise User
			rake db:migrate
		4. add route root and authenticate
			root 'statuses#index'
			#\app\controllers\statuses_controller.rb
  			before_action :authenticate_user!
		5. add view for show log_out and log_in
			# \app\views\layouts\application.html.erb

			 <% if user_signed_in? %>
			     <%= link_to "sign out",destroy_user_session_path, method: :delete   %>
			 <% end %>
			 <br>
		6. Setup cancan abilites
			rails g cancan:ability
			rails g model Role user_role:string
			# Role has_many :users
			# User belongs_to :role
			rails generate migration add_role_to_users role:references
			rake db:migrate
		6. add seed.rb
			Role.create!(user_role: 'admin');
			Role.create!(user_role: 'author');
			Role.create!(user_role: 'destroyer');
		7. Add role to model 
			# \app\models\user.rb
			  def has_role?(role)
			     roles == role
			   end

			   def roles
			     role.user_role.to_sym
			   end
		8. Add permited for devise in controller
			# \app\controller\application_controller.rb
				class ApplicationController < ActionController::Base
				  protect_from_forgery with: :exception
				  before_action :configure_permitted_parameters, if: :devise_controller?

				  private 

				  def configure_permitted_parameters
				    added_attrs = [:email, :password, :password_confirmation, :role_mask, :role_id]
				    devise_parameter_sanitizer.permit :sign_up, keys: added_attrs
				  end
				end
		9. Add new template for registration view
			#\app\views\devise\registrations\new.html.erb
				<%= devise_error_messages! %>
 

				  <div class="field">
				    <%= f.label :user_role %><br>
				    <%= f.collection_select :role_id, Role.all, :id, :user_role %>
				  </div>
		10. Change app model ability
			class Ability
			  include CanCan::Ability

			  def initialize(user)

			   user ||= User.new
			   if user.has_role? :admin
			     can :manage, :all
			   elsif user.has_role? :author
			     can :create, Status # author can create status
			     can :update, Status # author can update status
			     # can :destroy, Status # #uncomment this line, author can destroy status 
			     can :read, :all
			   else
			     can :read, :all
			   end
			  end
			end
		11. add loud_and_authorize resource
			# \app\controllers\statuses_controller.rb
			load_and_authorize_resource

