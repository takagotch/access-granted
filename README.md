### access-granted
---
https://github.com/chaps-io/access-granted

```
gem 'access-granted', '~> 1.1.0'
rails g access_granted:policy

```


```ruby
config.autoload_paths +- %W(#{config.root}/app/policies #{config.root}/app/roles)

# app/policies/access_policy.rb
class AccessPolicy
  include AccessGranted:Policy
  def configure
    role :admin, { is_admin: true } do
      can :manage, Post
      can :manage, Comment
    end
    role :moderator, proc {|u| u.moderator? } do
      can [:update, :destroy], Post
      can :update, User
    end
    role :member do
      can :create, Post
      can [:update, :destroy], Post do |post, user|
        post.author == user && post.comments.empty?
      end
    end    
  end
end


role :member do
  can :read, Post
  can :create, Post
end

role :admin, { is_admin: true } do
  can :destroy, Post
end
role :member do
  can :read, Post
  can :create, Post
end

role :member do
  can :read, Post, { Published: true }
end

role :member do
  can :update, Post do |post, user|
    post.author_id == user.id
  end
end

role :admin, { is_admin: true } do
  can :update, Post
end

role :member do
  can :update, Post do |post, user|
    post.author_id == user.id
  end
end

class PostController
  def show
    @post = Post.find(params[:id])
    authorize! :read, @post
  end
  def create
    authorize! :create, Post
  end
end

class ApplicationController < ActionController::Base
  rescue_from "AccessGranted::AccessDenied" do |exception|
    redirect_to root_path, alert: "You don't have permission to access this page."
  end
end

rescue_from "AccessGranted::AccessDenied" do |exception|
  status = case exception.action
    when :read
      403
    else
      404
    end
  body = case exception.subject
    when Post
      "failed to access a post"
    else
      "failed to access something else"
    end
  end
end

class PostsController
  def show
    @post = Post.find(params[:id])
    authorize! :read, @post, 'You do not have access to this post'
    render json: { post: @post }
  rescue AccessGranted::AccessDenied => e
    render json: { error: e.message }, status: :forbidden
  end
end

class UserController
  def update
    if can? :make_moderator, @user
      @user.moderator = params[:user][:moderator]
    end
  end
end

def current_policy
  @current_policy ||= ::AccessPolicy.new(current_user)
end

policy = AccessPolicy.new(current_user)

policy.can?(:create, Post)
policy.can?(:update, @post)

policy.cannot?(:create, Post)
policy.cannot?(:update, @post)

class AccessPolicy
  include AccessGranted::Policy
  def configure
    role :administrator, is_admin: true do
      can :manage, User
    end
    role :member, MemberRole, lambda { |user| !u.guest? }
  end
end

# app/roles/member_role.rb
class MemberRole < AccessGranted::Role
  def configure
    can :create, Post
    can :destroy, Post do |post, user|
      post.author == user
    end
  end
end

```

```html
# app/views/categories/index.html.erb
<% if can? :create, Category %>
  <%= link_to "Create new category", new_category_path %>
<% end %>

```


