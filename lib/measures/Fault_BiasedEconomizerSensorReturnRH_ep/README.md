# Biased Economizer Sensor: Return Relative Humidity

## Description

  def description
    return "When sensors drift and are not regularly calibrated, it causes a bias. Sensor readings often drift from their calibration with age, causing equipment control algorithms to produce outputs that deviate from their intended function. This fault is categorized as a fault that occur in the economizer system (sensor) during the operation stage. This fault measure is based on a physical model where certain parameter(s) is changed in EnergyPlus to mimic the faulted operation; thus simulates the biased economizer sensor (return relative humidity) by modifying Controller:OutdoorAir object in EnergyPlus assigned to the heating and cooling system. The fault intensity (F) is defined as the biased RH level (%). A positive number means that the sensor is reading a relative humidity higher than the true relative humidity."
  end
  
## Modeler Description

  def modeler_description
    return "Nine user inputs are required and, based on these user inputs, the return air RH reading in the economizer will be replaced by the equation below, RHraF = RHra + F*AF, where RHraF is the biased return air RH reading, RHra is the actual return air RH, F is the fault intensity and AF is the adjustment factor. To use this Measure, choose the Controller:OutdoorAir object to be faulted. Set the level of relative humidity sensor bias between -100 to 100 that you want at the return air duct for the economizer during the simulation period. For example, setting F=3 means the sensor is reading 25% when the actual relative humidity is 22%. The time required for the fault to reach the full level is only required when the user wants to model fault evolution. If the fault evolution is not necessary for the user, it can be defined as zero and F will be imposed as a step function with the user defined value. However, by defining the time required for the fault to reach the full level, fault starting month/date/time and fault ending month/date/time, the adjustment factor AF is calculated at each time step starting from the starting month/date/time to gradually impose F based on the user specified time frame. AF is calculated as follows, AF_current = AF_previous + dt/tau where AF_current is the adjustment factor calculated based on the previously calculated adjustment factor (AF_previous), simulation timestep (dt) and the time required for the fault to reach the full level (tau)."
  end
  
## Measure Type

EnergyPlus Measure

## Taxonomy

HVAC.HVAC Controls

## Arguments 

def arguments(workspace)
    args = OpenStudio::Ruleset::OSArgumentVector.new
    
    controlleroutdoorairs = workspace.getObjectsByType("Controller:OutdoorAir".to_IddObjectType)
    chs = OpenStudio::StringVector.new
    controlleroutdoorairs.each do |controlleroutdoorair|
      chs << controlleroutdoorair.name.to_s
    end
    econ_choice = OpenStudio::Ruleset::OSArgument::makeChoiceArgument('econ_choice', chs, true)
    econ_choice.setDisplayName("Choice of economizers.")
    econ_choice.setDefaultValue(chs[0].to_s)
    args << econ_choice
	
    ret_rh_bias = OpenStudio::Ruleset::OSArgument::makeDoubleArgument('ret_rh_bias', false)
    ret_rh_bias.setDisplayName('Enter the bias level of the return air relative humidity sensor. A positive number means that the sensor is reading a relative humidity higher than the true relative humidity. [%]')
    ret_rh_bias.setDefaultValue(-10)  #default fouling level to be -10%
    args << ret_rh_bias
	
    time_constant = OpenStudio::Ruleset::OSArgument::makeDoubleArgument('time_constant', false)
    time_constant.setDisplayName('Enter the time required for fault to reach full level [hr]')
    time_constant.setDefaultValue(0)  #default is zero
    args << time_constant
	
    start_month = OpenStudio::Ruleset::OSArgument::makeDoubleArgument('start_month', false)
    start_month.setDisplayName('Enter the month (1-12) when the fault starts to occur')
    start_month.setDefaultValue(6)  #default is June
    args << start_month
	
    start_date = OpenStudio::Ruleset::OSArgument::makeDoubleArgument('start_date', false)
    start_date.setDisplayName('Enter the date (1-28/30/31) when the fault starts to occur')
    start_date.setDefaultValue(1)  #default is 1st day of the month
    args << start_date
	
    start_time = OpenStudio::Ruleset::OSArgument::makeDoubleArgument('start_time', false)
    start_time.setDisplayName('Enter the time of day (0-24) when the fault starts to occur')
    start_time.setDefaultValue(9)  #default is 9am
    args << start_time
	
    end_month = OpenStudio::Ruleset::OSArgument::makeDoubleArgument('end_month', false)
    end_month.setDisplayName('Enter the month (1-12) when the fault ends')
    end_month.setDefaultValue(12)  #default is Decebmer
    args << end_month
	
    end_date = OpenStudio::Ruleset::OSArgument::makeDoubleArgument('end_date', false)
    end_date.setDisplayName('Enter the date (1-28/30/31) when the fault ends')
    end_date.setDefaultValue(31)  #default is last day of the month
    args << end_date
	
    end_time = OpenStudio::Ruleset::OSArgument::makeDoubleArgument('end_time', false)
    end_time.setDisplayName('Enter the time of day (0-24) when the fault ends')
    end_time.setDefaultValue(23)  #default is 11pm
    args << end_time

    return args
  end
  
## Initial Condition

    runner.registerInitialCondition("Imposing Sensor Bias on #{econ_choice}.")

## Final Condition

      runner.registerFinalCondition("Imposed Sensor Bias on #{econ_choice}.")

## Not Applicable

      runner.registerAsNotApplicable("#{name} is not running with zero fault level. Skipping......")
      runner.registerAsNotApplicable("MinimumFlowWithBypass in #{econ_choice} is not an economizer and is not supported. Skipping......")
      runner.registerAsNotApplicable(controlleroutdoorair.getString(14).to_s+" in #{econ_choice} is not supported. Skipping......")
      runner.registerAsNotApplicable(controlleroutdoorair.getString(25).to_s+" in #{econ_choice} is not supported. Skipping......")
      runner.registerAsNotApplicable("#{name} is not running for #{econ_choice} because of inapplicability. Skipping......")
	  
## Warning

n/a

## Error

      runner.registerError("Measure #{name} cannot find #{econ_choice}. Exiting......")

## Information

•	Measures below share the same resource codes.
•	BiasedEconomizerSensorReturnRH
•	BiasedEconomizerSensorOutdoorRH
•	Calculates required OA flow rate at given timestep. 
•	Heavy code. 
•	Works with, 
•	Controller:OutdoorAir.
Code Outline
•	Define arguments (economizer where fault occurs, fault level in constant value).
•	Find the economizer where the fault occurs.
•	Check applicability of the model to the economizer defined in the model.
•	Write EMS program (appropriately according to economizer options that were already defined in the model) to impose sensor bias for each economizer object.
•	Append EMS code to impose sensor bias level at the outdoor air measurement reading.
•	Append EMS code to recalculate other thermophysical properties based on biased reading.
•	Append appropriate EMS code if Minimum Outdoor Air Schedule option is defined.
•	Append appropriate EMS code if Mechanical Ventilation Controller option is defined.
•	Append appropriate EMS code if Economizer Control Type option is defined as NoEconomizer. If not,
•	Append appropriate EMS code if Lockout Type option is defined.
•	Append appropriate EMS code if Lockout Type option is defined as either LockoutWithHeating or LockoutWithCompressor.
•	Append appropriate EMS code if Economizer Control Type option is defined as DifferentialDryBulb.
•	Append appropriate EMS code if Economizer Control Type option is defined as either FixedDryBulb, FixedEnthalpy, FixedDewPointAndDryBulb or ElectronicEnthalpy.
•	Append appropriate EMS code if Economizer Control Type option is defined as DifferentialDryBulbAndEnthalpy.
•	Append appropriate EMS code if Economizer Control Type option is defined as DifferentialEnthalpy.
•	Append appropriate EMS code if Economizer Minimum Limit Dry-Bulb Temperature option is defined.
•	Append appropriate EMS code if High Humidity Control option is defined as yes.
•	Append appropriate EMS code if Control High Indoor Humidity Based on Outdoor Humidity Ratio option is defined as yes.
•	Append appropriate EMS code if Time of Day Economizer Control Schedule Name option is defined.
•	Append appropriate EMS code if Economizer Control Action Type option is defined as MinimumFlowWithBypass.
•	Append appropriate EMS code if High Humidity Control option is defined as yes.
•	Append appropriate EMS code if Minimum Fraction of Outdoor Air Schedule Name option is defined.
•	Append appropriate EMS code if Maximum Fraction of Outdoor Air Schedule Name option is defined.
•	Append appropriate EMS code to calculate modified outdoor air flow rate.
•	Append appropriate EMS code to check whether modified outdoor air flow rate exceeds maximum limit.
•	Append appropriate EMS code and texts for defining objects in idf based on above options to complete the code.
•	Define EnergyManagementSystem:Subroutine
•	Define EnergyManagementSystem:ProgramCallingManager
•	Define EnergyManagementSystem:GlobalVariable
•	Define EnergyManagementSystem:Actuator
•	Define EnergyManagementSystem:InternalVariable
•	Define EnergyManagementSystem:Sensor
•	Define Output:EnergyManagementSystem
•	Append EMS code that calculates the adjustment factor (AF)

## Tests

●	Test model with different Cotroller:OutdoorAir options.
●	Test invalid user argument values to make sure measure fails gracefully.



