---
layout: post
title:      "UEFA - Schedules (Rails w JavaScript App)"
date:       2018-12-02 20:12:17 -0500
permalink:  uefa_-_schedules_rails_w_js_app
---


For this project I decided to build a new Rails app instead of using my previous Rails project app as suggested, mainly because I wanted to keep my Rails and Ruby skills sharp, but also because we had just learned how to consume and build API applications. I figured I would create a Rails app that would consume a third party API data and use that data to render information instead of relying on using databases as we had done on the previous project. I am glad I made this decision because as challenging as it was to finish this project, it also taught me a lot about how the internet works and about general web development architecture.

Since my new app was not going to be a typical CRUD app, and therefore I was not going to use a database and instead use API data, the first challenge was to develop a system to request information and process it. I also had to render this information as JSON objects so that the JavaScript front-end (or other apps) could fetch this information from my back-end. So I started by building the routes and endpoints for what I was panning to render. Most of them we custom routes but still following RESTful pricinciples:

```
  root 'welcome#home'

  resources :leagues, only: [:index, :show]
  get '/leagues/:id/matches/current', to: 'leagues#current_matches'
  get '/leagues/:id/matches/scheduled', to: 'leagues#scheduled_matches', as: 'league_scheduled_matches'
  get '/leagues/:id/matches/all', to: 'leagues#all_matches', as: 'league_all_matches'
  get 'leagues/:id/teams', to: 'leagues#teams', as: 'league_teams'

  get 'teams/:id', to: 'teams#show'

  get '/today', to: 'todays#today'

  resources :reviews, only: [:index, :create]
end
```

Next I built the controllers. Initially I placed all the logic in them but quickly realized that controllers shouldn’t be responsible for requesting external data, processing the data and rendering the data. So to follow the single responsibility principle, I created service objects to communicate with the third party API, process the responses, and then feed those to the controller. I approached the design of the service object to function in a similar way as Active Record, that is have the ability for the controller to find data by id or parameters through the service objects. The three main data “objects” the application is concerned with are, Leagues, Matches, and Teams: 

```
  def self.find_league(id)
    url = "https://api.football-data.org/v2/competitions/#{id}"
    resp = Faraday.get url do |req|
      req.headers['X-Auth-Token'] = ENV['AUTH_TOKEN']
    end

    body = JSON.parse(resp.body)

    if resp.success?
      @response = body
     else
      @response = "ERRORS"
    end
    @response

  end

  def self.get_matches(league, query="")
    url = "https://api.football-data.org/v2/competitions/#{league}/matches?#{query}"
    resp = Faraday.get url do |req|
      req.headers['X-Auth-Token'] = ENV['AUTH_TOKEN']
    end

    body = JSON.parse(resp.body)

    if resp.success?
      @response = body["matches"]
      else
      @response = "ERRORS"
    end
    @response
  end


    def self.get_teams(id)
      url = "https://api.football-data.org/v2/competitions/#{id}/teams"
      resp = Faraday.get url do |req|
        req.headers['X-Auth-Token'] = ENV['AUTH_TOKEN']
      end

      body = JSON.parse(resp.body)

      if resp.success?
        @response = body
        else
        @response = "ERRORS"
      end
      @response
    end

end
```


This leagues service object corresponds to the leagues controller. Each controller in the app has a corresponding service object. Once I made sure the connections were set on the service object , the next step on the data flow was to to call the service object in the controllers to process the data. This is an example of on of the leagues controller actions calling on the service object:

```
  def current_matches
    @league = LeagueService.find_league(params[:id])
      if params[:matchday]
        @matchday = params[:matchday].to_i
      else
        @matchday = @league["currentSeason"]["currentMatchday"] 
      end
      @query = "matchday=#{@matchday}"
      @league_name = @league["name"]
      @matches = LeagueService.get_matches(params[:id], @query)
        if @matches != "ERRORS"
          json = @league.to_json
          respond_to do |format|
          format.html { render :current_matches}
          format.json { render json: json }
          end
        else
          respond_to do |format|
            format.html { render "welcome/errors", layout: false }
            format.json { render json: {status: "error", code: 429, message: "Too many requests. Please try again in one minute"} }
            end
      end 
  end
	```
	
This made it easier and more practical to render data and manage errors. 

After this was setup and the rails app was fully functional, the next step was to use the JSON serialization objects to send responses from the JavaScript AJAX calls. Requesting the JSON objects and processing the information is very straight forward. On some occasions I used `fetch()`, in other I used jQuery `$.post()` or `$.get()`. Only so I could practice and learn both ways. No particular preference. One AJAX method I discovered was the jQuery `.load()`. This makes it very easy to extract html from elements in other pages outside the current one in one line of code:



`$('.current-matches').load(`/leagues/${this.dataset.id}/matches/current .matches-table`)`



The hardest part about rendering AJAX JS was the actual rendering of the JSON objects into the html pages. I was working mainly with html forms with several rows and headers, so at times it was challenging to populate or append the html in them. I didn't wanted to use Handlebars or other templating resources, instead I used ES6 template literals to do this, which was a bit tedious sometimes.

Overall I really enjoyed working on this project. I was forced to master my developer googling skills and therefore learned plenty!

GitHub Repo:
https://github.com/asabogal/UEFA-schedules






