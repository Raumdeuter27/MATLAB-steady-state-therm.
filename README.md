thermalmodel = createpde("thermal","steadystate-axisymmetric");
g = decsg([3 4 0 0 .2 .2 -1.5 1.5 1.5 -1.5]');
geometryFromEdges(thermalmodel,g);
%% 
figure
pdegplot(thermalmodel,"EdgeLabels","on")
axis equal
%% 
k = 130; % Thermal conductivity, W/(m*C)
rho = 2810; % Density, kg/m^3
cp = 960; % Specific heat, W*s/(kg*C)
q = 20000; % Heat source, W/m^3
%% 
thermalProperties(thermalmodel,"ThermalConductivity",k);
internalHeatSource(thermalmodel,q);
thermalBC(thermalmodel,"Edge",2,"Temperature",100);
thermalBC(thermalmodel,"Edge",3,...
                       "ConvectionCoefficient",50,...
                       "AmbientTemperature",100);
thermalBC(thermalmodel,"Edge",4,"HeatFlux",5000);
%% 
msh = generateMesh(thermalmodel);
figure
pdeplot(thermalmodel)
axis equal
%% 
result = solve(thermalmodel);
T = result.Temperature;
figure
pdeplot(thermalmodel,"XYData",T,"Contour","on")
axis equal
title("Steady-State Temperature")
%%
%%transient
thermalmodel.AnalysisType = "transient-axisymmetric";
thermalProperties(thermalmodel,"ThermalConductivity",k,...
                               "MassDensity",rho,...
                               "SpecificHeat",cp);
%% 
thermalIC(thermalmodel,0);
tfinal = 50000;
tlist = 0:100:tfinal;
result = solve(thermalmodel,tlist);
%% 
T = result.Temperature;

figure 
pdeplot(thermalmodel,"XYData",T(:,end),"Contour","on")
axis equal
title(sprintf(['Transient Temperature' ...
               ' at Final Time (%g seconds)'],tfinal))
               %% 
Tcenter = interpolateTemperature(result,[0.0;-1.5],1:numel(tlist));
Touter = interpolateTemperature(result,[0.2;-1.5],1:numel(tlist));               
%% 
figure
plot(tlist,Tcenter)
hold on
plot(tlist,Touter,"--")
title("Temperature at the Bottom as a Function of Time")
xlabel("Time, s")
ylabel("Temperature, C")
grid on
legend("Center Axis","Outer Surface","Location","SouthEast")
