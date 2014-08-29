# Models of Segregation

## Simulation Code

We start with our agents.  Our agents exist in a 2D space and have a strategy.  For now, we will randomly assign them their strategy and geographic location. 


		class Agent
			constructor: (@space) ->
				@strategy = switch Math.floor Math.random() * 8
					when 0 then {name: "All Defect", color: "red", i: 0, c: 0, d: 0}
					when 1 then {name: "Suspicious Perverse", color: "blue", i: 0, c: 0, d: 1}
					when 2 then {name: "Suspicious Tit-for-Tat", color: "midnightblue", i: 0, c: 1, d: 0}
					when 3 then {name: "D-then-All-Cooperate", color: "green", i: 0, c: 1, d: 1}
					when 4 then {name: "C-then-All-Defect", color: "lime", i: 1, c: 0, d: 0}
					when 5 then {name: "Perverse", color: "yellow", i: 1, c: 0, d: 1}
					when 6 then {name: "Tit-for-Tat", color: "orange", i: 1, c: 1, d: 0}
					when 7 then {name: "All Cooperate", color: "indigo", i: 1, c: 1, d: 1}

				@x = Math.floor Math.random() * @space.width
				@y = Math.floor Math.random() * @space.height
				@step 	= 10 #@space.width * @space.height / 7200
				@depth 	= 50
				@last_action = null
				@score = 0




Next, we difine a 2D space representing the problem domain. Our space contains a geographical distribution of agents stored in a list.  We also give a space a width and height to manage the relation between the simulation and the client.


		class Space
			constructor: (@height, @width) ->
				@agents = []


In spacial arranements, everybody is next to somebody - their neighbour.  The relative composition of that neighbourhood - 10% homogeniality or 50% homogeniality - is determined by the race of ones neighbours and how deep the conception of neighbourhood extends.


			neighbourhood: (x, y) ->
				neighbours = []
				for other in @agents
					if x - other.depth < other.x < x + other.depth and y - other.depth < other.y < y + other.depth
						neighbours.push other
				neighbours


Now that we have defined our model, we need some functions to initiate and control behaviour.  We will instantiate the simulation by invoking the `agents` function.  This will create a space and populate it with agents.


		agents = (height, width) ->
			space = new Space(height, width)
			for n in [1..2000]
				space.agents.push new Agent space 
			space.agents


We then need some logic for moving our agents around the space.  Movement could be intentionally directed or (as in this case), agents move to a random spot near by, only moving if they are unhappy with their neighbourhood.


		move = (agent) ->
			xRand = Math.random() * agent.step
			yRand = Math.random() * agent.step
			agent.x += xRand - agent.step / 2
			agent.x = xRand / 2 if agent.x < 0
			agent.x = agent.space.width - xRand /2 if agent.x > agent.space.width
			agent.y += yRand - agent.step / 2
			agent.y = yRand /2 if agent.y < 0
			agent.y = agent.space.height - yRand / 2 if agent.y > agent.space.height



		prisoners_dilemma = (player1, player2) ->
			return [3, 3] if player1.last_action is 1 and player2.last_action is 1
			return [0, 5] if player1.last_action is 1 and player2.last_action is 0
			return [5, 0] if player1.last_action is 0 and player2.last_action is 1
			return [1, 1] if player1.last_action is 0 and player2.last_action is 0


Our new API action is play.  The rough idea is to get everybody in the neighbourhood together, compete against each, and update strategy depending on which is the most successful in the neighbourhood.


		play = (agent) ->
			neighbours = agent.space.neighbourhood(agent.x, agent.y)
			others = []
			for neighbour in neighbours
				[agent, neighbour] = compete agent, neighbour
				others.push neighbour
			for neighbour in neighbours
				agent.strategy = neighbour.strategy unless agent.score >= neighbour.score
			agent


		compete = (agent, neighbour) ->
			agent.act neighbour
			neighbour.act agent
			scores =  prisoners_dilemma agent, neighbour
			agent.score = scores[0]
			neighbour.score = scores[0]
			[agent, neighbour]


An agent takes an action based on either their initial action or 

		Agent.prototype.act = (neighbour) ->
			@last_action = if @last_action? then @respond neighbour else @strategy.i

		Agent.prototype.respond = (neighbour) ->
			if neighbour.action is 1 then @strategy.c else @strategy.d

		Agent.prototype.action = () ->
			if @last_action? then last_action else @strategy.i




- interact() 
- if last_action.nil? then initial_move else coo


Finally, we declare our public API so that other modules can access it.


		module.exports = {agents: agents, move: move, play: play}