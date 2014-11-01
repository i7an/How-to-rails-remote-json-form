#HowTo: Rails remote json form

Usually there is one page with two or more forms. And it causes pain in ass when I trying to implement it. I used to set `remote: true` and render `create.js.erb` after. But I faced a new problem: i need to render different forms with errors if creation failed on different pages. I understood that old way is wrong (finaly ahaha).

##Solution

###View

    <%= simple_form_for user, remote: true, format: :json do |f| %>
        <%= f.input :email %>
        <%= f.input :first_name %>
        <%= f.input :last_name %>

        <div class="form_nav">
          <%= f.button :submit %>
        </div>
    <% end %>

### Controller

    def create
      @user = User.new(user_params)
      respond_to do |format|
        if @user.save
          format.html { redirect_to @user, notice: 'Success' }
          format.json { render json: { user: @user } }
        else
          format.html { render 'new' }
          format.json { render json: { errors: @user.errors }, status: 422 }
        end
      end
    end
    
### Coffee

#### Super-class

    class @RemoteJsonForm
      constructor: (form) ->
        @form = $(form)
        @form.bind('ajax:success', (xhr, data) =>
          @resetForm()
          @onSuccess(data)
        ).bind('ajax:error', (xhr, data) =>
          @_clearErrors()
          errors = data.responseJSON.errors
          @_addErrors(errors)
          @onError(errors)
        )
    
      _addErrors: (errorsObj) ->
        for name, errors of errorsObj
          $("##{@resourceName()}_#{name}", @form).parent('.input_wrap').addClass('error').after @_errorsHtml(errors)
    
      _clearErrors: ->
        $('.input_wrap.error', @form).removeClass('error').nextAll('.error_mess').remove()
    
      _errorsHtml: (errors) ->
        join = (a, b) -> a + b
        _.reduce(_.map(errors, (error) -> "<div class=\"error_mess\">#{error}</div>"), join, '')
    
      resetForm: ->
        @_clearErrors()
        @form[0].reset()
    
      resourceName: -> throw 'Not implemented'
      onSuccess: (data) -> # nop
      onError: (errors) -> # nop
      
#### Implementation

    class UserForm extends RemoteJsonForm
      constructor: (form, uberObj) ->
        @uberObj = uberObj # just for example
        super(form)
    
      onSuccess: (data) ->
        user = data.user
        uberObj.add(user) # just for example
    
      resourceName: -> 'user'
      
What you left to do is to create a new instance `new UserForm('.uber-new-user')`. You can create many implementations of form base class with needed logic.
