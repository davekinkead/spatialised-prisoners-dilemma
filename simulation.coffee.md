# Models of Segregation

## Simulation Code


First, let's define some aspects of a Prisoner's Dilemma.  A PD is a two (or more) player symmertic non-cooperative non-zero sum game that has payoffs for each player based on the behaviour of other players.  We will create a generic function that returns a tuple representing conditional player outcomes.


		prisoners_dilemma = (player1, player2) ->
			return [3, 3] if player1.last_action is 1 and player2.last_action is 1
			return [0, 5] if player1.last_action is 1 and player2.last_action is 0
			return [5, 0] if player1.last_action is 0 and player2.last_action is 1
			return [1, 1] if player1.last_action is 0 and player2.last_action is 0


There are 8 possible deterministic single round strategies a player could employ.  These are specified by their inital move `i`, responding to cooperation move `c`, and responding to defection move `d`.  We'll also name these and give them pretty colours and store them in a list.


		strategies = [
			{ i: 0, c: 0, d: 0, name: "ALLD", color: "red" },
			{ i: 0, c: 0, d: 1, name: "SPRV", color: "green" },
			{ i: 0, c: 1, d: 0, name: "ST4T", color: "blue" },
			{ i: 0, c: 1, d: 1, name: "DTAC", color: "indigo" },
			{ i: 1, c: 0, d: 0, name: "CTAD", color: "yellow" },
			{ i: 1, c: 0, d: 1, name: "PERV", color: "lime" },
			{ i: 1, c: 1, d: 0, name: "FT4T", color: "skyblue" },
			{ i: 1, c: 1, d: 1, name: "ALLC", color: "violet" }
		]


Next we model our agents.  Our agents exist in a 2D space and have a strategy, a score, and a last_action to keep track of behavior and performance.  For now, we will randomly assign them their strategy and geographic location.


		class Agent
			constructor: (@space) ->
				@strategy = strategies[Math.floor Math.random() * 8]
				@x = Math.floor Math.random() * @space.width
				@y = Math.floor Math.random() * @space.height
				@step = 15
				@last_action = null
				@score = 0



Agents live in a space. We difine a 2D space representing the problem domain. Our space contains a geographical distribution of agents stored in a list.  We also give a space a width and height to manage the relation between the simulation and the client.


		class Space
			constructor: (@height, @width) ->
				@agents = []
				@depth = 25


In spacial arranements, everybody is next to somebody - their neighbour.  A neighbourhood is simply a list of all the agents within an agent's depth perception.  Here we return everyone within a square from an x, y coordinate.


			neighbourhood: (x, y) ->
				neighbours = []
				for other in @agents
					if x - other.space.depth < other.x < x + other.space.depth and y - other.space.depth < other.y < y + other.space.depth
						neighbours.push other
				neighbours


Now we turn to our game.  For every agent, we get all their neighbours, the play our game against them with `compete`, before updating the player's strategy to one of the equal best strategies employed by her neighbours.  We also throw in some Brownian motion to encourage disequilibrium.


		play = (agent) ->
			agent.score = 0
			neighbours = agent.space.neighbourhood(agent.x, agent.y)
			for neighbour in neighbours
				agent = compete agent, neighbour
			for neighbour in neighbours
				agent.strategy = neighbour.strategy unless agent.score >= neighbour.score

			move agent
			agent


While an agent plays against everyone in their neighbourhood, a game applies between just two agents.  Here, we get an agent to act against her neighbour, and update their scores based on the strategy employed. We play the prisoner's dilemma n times for each neighbour.


		compete = (agent, neighbour) ->
			agent.act neighbour
			neighbour.act agent
<<<<<<< HEAD
			scores = []
			agent_total = 0
			neighbour_total = 0
=======
>>>>>>> upstream/master
			for n in [0..20]
				scores = prisoners_dilemma agent, neighbour
				agent.score += scores[0]
			agent


We now need to extend the functionality of the Agent class.  In a game, an agent takes an action based on either their initial action if it's their first, or the responce to the other players cooperation or defection.

		Agent.prototype.act = (neighbour) ->
			@last_action = if @last_action? then @respond neighbour else @strategy.i

		Agent.prototype.respond = (neighbour) ->
			if neighbour.action is 1 then @strategy.c else @strategy.d

		Agent.prototype.action = () ->
			if @last_action? then last_action else @strategy.i


Let's also throw in some logic for moving our agents around the space.  Movement could be intentionally directed or (as in this case), agents move to a random spot near by.


		move = (agent) ->
			xRand = Math.random() * agent.step
			yRand = Math.random() * agent.step
			agent.x += xRand - agent.step / 2
			agent.x = xRand / 2 if agent.x < 0
			agent.x = agent.space.width - xRand /2 if agent.x > agent.space.width
			agent.y += yRand - agent.step / 2
			agent.y = yRand /2 if agent.y < 0
			agent.y = agent.space.height - yRand / 2 if agent.y > agent.space.height


Now that we have defined our model, we need some functions to initiate and control behaviour.  We will instantiate the simulation by invoking the `agents` function.  This will create a space and populate it with agents.


		agents = (height, width) ->
			space = new Space(height, width)
			for n in [1..500]
				space.agents.push new Agent space
			space.agents


Finally, we declare our public API so that other modules can access it.


		module.exports = {agents: agents, move: move, play: play}


That's it. Simulation complete!
