1. Articles
    
         + https://blog.joshsoftware.com/2014/05/08/implementing-rails-apis-like-a-professional/
         + https://sourcey.com/building-the-prefect-rails-5-api-only-app/
         https://scotch.io/tutorials/build-a-restful-json-api-with-rails-5-part-one
         
2. How to generate app only for api?
        
        rails new my_app --api
         
## SECURITY
2. How to add method for authorization api through token?
        
        # Before action 
        before_action :authenticate
        
        # this block should return true or false
        def authenticate
            authenticate_or_request_with_http_token |token,other_options|
             Apiclient.find_by_client_key(token).present?
            end
        end
        
3. In what place you have to put Api key?
        
         API key in the request header 
4. How you can take API key from request header?
        
        # Before action 
        before_action :authenticate 
        
        def authenticate
          api_key = request.headers['X-Api-Key']
          @user = User.where(api_key: api_key).first if api_key

          unless @user
            head status: :unauthorized
            return false
          end
        end
5. How to set/get Rails app secrets?
        
        # config/secrets.yml
        development:
          secret_key_base: ec....
          my_own_token: aw...
        
        # call it
        Rails.application.secrets.secret_key_base
6. Gem for protect your web app from bad clients
        
        https://github.com/kickstarter/rack-attack
7. How to check your access to website throught a token in command line?
        
        curl -H "Authorization: Token token=lIQAaDgj8Sl5GREFYDtgEwtt" http://api.lvh.me:3000/v1/users