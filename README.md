# Mesa SIR

Mesa SIR is an extension for Python's Agent Based Modeling Library Mesa. Mesa SIR provides the basic building blocks for an Agent Based Susceptible-Infected-Recovered (SIR) Epidemic model. The hope is others will improve upon it to make it a robust ABM extension to aid in understanding and decision making for both COVID-19 and future pandemics.

A full model version is available [COVID MESA ABM](https://github.com/metalcorebear/COVID-Agent-Based-Model)

***Please contribute to help make this extension better. Some ways to contribute are:*** 
* Add the ability to bring in GIS extensions with [Mesa Geo](https://github.com/Corvince/mesa-geo)
* Make more complex populations
* Make more complex social dynamics
* Create Mesa MultiGrid option
* Add BatchRunner option
 


## Requirements

Mesa SIR requires

    Mesa>=0.8.6
    numpy>=1.16.5
	networkx >=2.3
	matplotlib>=3.1.1


## Installation 

    pip install mesa-SIR

    pip install git+https://github.com/metalcorebear/Mesa-SIR.git


Currently, Mesa SIR has two modules. 

## SIR Module

The main module is the SIR.py module. The SIR modules has two main parts. 

### Build Network method 

The build network class has two parameters:

	interactions = number of agents each agent will interact with each time step; integer
	population = number of agents; integer 

Using Mesa's NetworkGrid class this module builds a network of agents and their interactions and can be instantiated as follows. 

	from mesa_SIR import SIR

	# returns a networkx graph to place in your Mesa NetworkGrid instance
	G = SIR.build_network(interactions, population)
    self.grid = NetworkGrid(G)

### Infection Class

The infection class has two main part interactions and infection.

The infection class has the following key word arguments (kwargs):

- ptrans = 0.25 #probability of transmission; float between 0 and 1 
- reinfection_rate = 0.00 #probability of reinfection; float between 0 and 1
- I0 =0.10 # initial infection rate; float between 0 and 1
- severe =0.18 # probability of severity; float between 0 and 1
- progression_period = 3 #number of days for disease to progress and show symptoms-- mean of a Gaussian distribution
- progression_sd = 2 #standard deviation of disease progression -- sd of a Gaussian distribution 
- death_rate = 0.0193 #probability of disease being fatal
- recovery_days = 21 #number of days for disease to complete -- mean of Gaussian distribution
- recovery_sd = 7 #standard deviation of disease to finish -- sd of Gaussian distribution


The infection class can be instantiated as follows in the model initiation: 

	self.SIR_instance = SIR.Infection(self, ptrans = 0.40,
                                          reinfection_rate = 0.10,
                                          I0= 0.02,
                                          severe = 0.90,
                                          progression_period = 5,
                                          progression_sd = 2,
                                          death_rate = 0.30,
                                          recovery_days = 38,
                                          recovery_sd = 6

Due to the requirements of the infection model an agent needs several attributes. An example agent class instantiation with the required attributes is as follows. This initiation uses the Infection class' initial_infection method.

	class human(Agent):
    
    def __init__(self, unique_id, model):
        super().__init__(unique_id, model)
        self.pos = unique_id
        # Infection class initial infection method
        self.was_infected = False
        self.infected, self.susceptible, self.severe = self.model.SIR_instance.initial_infection() 
        self.recovered = False
        self.alive = True
        self.day = 0
        self.induced_infections = 0
        self.infected_others = False

Agent interaction and infection are then executed in the Infection class interaction method as follows: 

	    def step(self):

        	self.model.SIR_instance.interact(self)
        	self.day += 1


## Calculations and Plots Module

This module conducts several computations and produces plots and data for the model. 

An example of how to call the collections and plots modules and use with Mesa's datacollection module is as follows. 

During the model initiation phase: 

	from mesa_SIR import calculations_and_plots as c_and_p


	self.datacollector = DataCollector(model_reporters={"infected": lambda m: c_p.compute(m,'infected'),
                                                            "recovered": lambda m: c_p.compute(m,'recovered'),
                                                            "susceptible": lambda m: c_p.compute(m,"susceptible"),
                                                            "dead": lambda m: c_p.compute(m, "dead"),
                                                            "R0": lambda m: c_p.compute(m, "R0"),
                                                            "severe_cases": lambda m: c_p.compute(m,"severe")})

To save the dataframe and conduct the plots after the model is complete: 

	output_data = SIR_model.datacollector.get_model_vars_dataframe()
	
	#Save the model dataframe
	c_p.save_data(output_data) optional keywords: output_path, filename
	
	#Create and save plots
	title = 'SIR ABM Model Output'
	c_p.plot_SIR(output_data, title) optional keyword: output_path
	c_p.plot_R0(output_data, title) optional keyword: output_path
	c_p.plot_severe(output_data, title) optional keyword: output_path










