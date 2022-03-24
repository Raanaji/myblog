---
template: BlogPost
path: /prac5
mockup: 
thumbnail:
date: 2019-03-25
name: Computing price of a  European Option using Crank-Nicholson Method
category: Quantitative Finance
description: 'Using MATLAB for exotic options valuation.'
tags:
  - Matlab 
  - crank nicholson
  - computational finance
---

Computing the price of a  European Call and Put Option by Implicit and Crank-Nicholson Method

We start by computing the prices of European Put option under GBM with the parameters below, for different strike prices **K = 80, 90, 100, 110, 120**. Using the homogeneous property of the option prices with respect to stock price and strike, namely,

**S(0) = 100,r = 0.03,q = 0.05,σ = 0.2,T = 1.**

**P (λS(0), λK) = λP (S(0), K), ∀λ > 0;**

where **P (S(0), K)** is the European put option price at time **0** with S(0) and **K** being the initial stock price and strike, respectively of the put option.

Let us develop the code to implement the Crank-Nicolson scheme to compute the prices of European call option, using the recursive CrankNicolson discretization scheme in matrix form

```matlab
% CrankNicolsonFDM_BS_Eur_Price_Put.m
%%
%error = [];
%%
% Implementing the Crank-Nicolson Scheme to solve the BS PDE with constant
% volatility
% Strike price
K = 1;
% Set the minimal and maximal stock prices
Smin = 0;
Smax = 4*K;

% The number of points in stock direction
N = 10000;
S1 = linspace(Smin,Smax,N+1)';
% The length of the stock price interval
dS = S1(2) - S1(1);
% S stores all the prices except boundary points
S = S1(2:N);

% Maturity of option
T = 1;
% The number of points in time dimension
M = 100;
tau = linspace(0,T,M+1);
% The length of the time interval
dtau = tau(2) - tau(1);

% Interest rate
r = 0.03;
% Dividend yield
q = 0.05;
% Volatility (Constant)
sigma = 0.2;

% alpha and beta in the discretization scheme
alpha = 0.5*sigma^2*S.^2*dtau/(dS^2);
beta = (r - q)*S*dtau/(2*dS);

% lower, main and upper diagonal of the tridiagonal matrix for the implicit
% finite difference scheme
l = -alpha + beta;
d = 1 + r*dtau + 2*alpha;
u = -(alpha + beta);

% lower, main and upper diagonal of the tridiagonal matrix for the explicit
% finite difference scheme
lE = alpha - beta;
dE = 1 - r*dtau - 2*alpha;
uE = alpha + beta;

%AE = diag(d,0)+diag(l(2:end),-1)+diag(u(1:end-1),1);
% Vector to store the option prices at time k
% Option payoff at maturity
Vold = max(K - S,0);
% Vector to store the option prices at time k+1
Vnew = Vold;
temp = Vold;
for k=1:M
    % Dirichlet Boundary condition for European put option at time k+1
    boundary1 = [-l(1)*K*exp(-r*dtau);zeros(N-3,1);-u(N-1)*0];
    % Dirichlet Boundary condition for European put option at time k
    boundary = [lE(1)*K*exp(-r*(k-1)*dtau) - l(1)*K*exp(-r*k*dtau);zeros(N-3,1);uE(N-1)*0 - u(N-1)*0];
    % To make sure 2nd order convergence in time, a couple of implicit
    % scheme is implemented before switching to crank nicolson scheme
    if(k<1)
        Vnew = tridiag(d,u,l,Vold+boundary1);
    else    
    % Crank Nicolson iteration scheme
    % Remember the correct scheme is that A+I, hence instead of dE, we need
    % to use dE+1
    for j=1:N-1
        if(j==1)
            temp(j) = (dE(j)+1)*Vold(j) + uE(j)*Vold(j+1);
        elseif(j<N-1)
            temp(j) = lE(j)*Vold(j-1) + (dE(j)+1)*Vold(j) + uE(j)*Vold(j+1);
        else
            temp(j) = lE(j)*Vold(j-1) + (dE(j)+1)*Vold(j);
        end
    end
    %temp = tridiag_prod(dE+1,uE,lE,Vold);
    Vnew = tridiag(d+1,u,l,temp+boundary);
    end
    % Update the vectors from time k to time k+1
    Vold = Vnew;
end
% Interpolation to find the put price when S0 = 1.0
S0 = 1.0;
put_fdm = interp1(S,Vold,S0);
%error = [];
% Compare the prices with the BS formula
[call_bs, put_bs] = blsprice(S0,K,r,T,sigma,q);
error = [error; [M, abs(put_bs - put_fdm)]];

%%
% Plot the prices from both the Crank-Nicolson method and the BS formula
[call_bs, put_bs] = blsprice(S,K,r,T,sigma,q);
subplot(2,1,1)
plot(S,Vold,S,put_bs,'LineWidth',2)
title('European Put price, Crank-Nicolson - BS formula')
xlabel('Stock price')
ylabel('Put price')
legend('Crank-Nicolson','BS formula','Location','SouthEast')
subplot(2,1,2)
plot(S,Vold-put_bs,'LineWidth',2)
title('Difference of Put price, Crank-Nicolson - BS formula')
xlabel('Stock price')
ylabel('Put price')
```

```matlab
% CrankNicolsonFDM_BS_Eur_Price.m
%%
% Implementing the Crank-Nicolson Scheme to solve the BS PDE with constant
% volatility

% Strike price
K = 1;
% Set the minimal and maximal stock prices
Smin = 0;
Smax = 4*K;

% The number of points in stock direction
N = 5000;
S1 = linspace(Smin,Smax,N+1)';
% The length of the stock price interval
dS = S1(2) - S1(1);
% S stores all the prices except boundary points
S = S1(2:N);

% Maturity of option
T = 1;
% The number of points in time dimension
M = 5000;
tau = linspace(0,T,M+1);
% The length of the time interval
dtau = tau(2) - tau(1);

% Interest rate
r = 0.03;
% Dividend yield
q = 0.05;
% Volatility (Constant)
sigma = 0.2;

% alpha and beta in the discretization scheme
alpha = 0.5*sigma^2*S.^2*dtau/(dS^2);
beta = (r - q)*S*dtau/(2*dS);

% lower, main and upper diagonal of the tridiagonal matrix for the implicit
% finite difference scheme
l = -alpha + beta;
d = 1 + r*dtau + 2*alpha;
u = -(alpha + beta);

% lower, main and upper diagonal of the tridiagonal matrix for the explicit
% finite difference scheme
lE = alpha - beta;
dE = 1 - r*dtau - 2*alpha;
uE = alpha + beta;

%AE = diag(d,0)+diag(l(2:end),-1)+diag(u(1:end-1),1);
% Vector to store the option prices at time k
% Option payoff at maturity
Vold = max(S - K,0);
% Vector to store the option prices at time k+1
Vnew = Vold;
temp = Vold;
for k=1:M
    % Dirichlet Boundary condition for European call option at time k+1
    boundary_I = [-l(1)*0;zeros(N-3,1);-u(N-1)*(Smax*exp(-q*k*dtau)-K*exp(-r*k*dtau))];
    % Dirichlet Boundary condition for European call option at time k
    boundary = [l(1)*0;zeros(N-3,1);uE(N-1)*(Smax*exp(-q*(k-1)*dtau)-K*exp(-r*(k-1)*dtau))-...
        u(N-1)*(Smax*exp(-q*k*dtau)-K*exp(-r*k*dtau))];
    %boundary = [l(1)*0;zeros(N-3,1);uE(N-1)*0];
    % To make sure 2nd order convergence in time, a couple of implicit
    % scheme is implemented before switching to crank nicolson scheme
    if(k<3)
        Vnew = tridiag(d,u,l,Vold+boundary_I);
    else    
    % Crank Nicolson iteration scheme
    % Remember the correct scheme is that A+I, hence instead of dE, we need
    % to use dE+1
    % Explicit iteration scheme
    for j=1:N-1
        if(j==1)
            temp(j) = (dE(j)+1)*Vold(j) + uE(j)*Vold(j+1);
        elseif(j<N-1)
            temp(j) = lE(j)*Vold(j-1) + (dE(j)+1)*Vold(j) + uE(j)*Vold(j+1);
        else
            temp(j) = lE(j)*Vold(j-1) + (dE(j)+1)*Vold(j);
        end
    end
    %temp = tridiag_prod(dE+1,uE,lE,Vold);
    Vnew = tridiag(d+1,u,l,temp+boundary);
    end
    % Update the vectors from time k to time k+1
    Vold = Vnew; 
end
% Interpolation to find the call price when S0 = 1.0
S0 = 1.0;
call_fdm = interp1(S,Vold,S0);
%error = [];
% Compare the prices with the BS formula
[call_bs, put_bs] = blsprice(S0,K,r,T,sigma,q);
%error = [error; [M, abs(call_bs - call_fdm)]];

%%
% Plot the prices from both the Crank-Nicolson method and the BS formula
subplot(2,1,1)
plot(S,Vold,S,blsprice(S,K,r,T,sigma,q),'LineWidth',2)
title('European Call price, Crank-Nicolson - BS formula')
xlabel('Stock price')
ylabel('Call price')
legend('Crank-Nicolson','BS formula','Location','SouthEast')
subplot(2,1,2)
plot(S,Vold-blsprice(S,K,r,T,sigma,q),'LineWidth',2)
title('Difference of Call price, Crank-Nicolson - BS formula')
xlabel('Stock price')
ylabel('Call price')
```

```matlab
% ImplicitFDM_BS_Eur_Price_Put.m
% Implementing the Implicit Scheme to solve the BS PDE with constant
% volatility
% Strike price
K = 1;
% Set the minimal and maximal stock prices
Smin = 0;
Smax = 4*K;

% The number of points in stock direction
N = 800;
S1 = linspace(Smin,Smax,N+1)';
% The length of the stock price interval
dS = S1(2) - S1(1);
% S stores all the prices except boundary points
S = S1(2:N);

% Maturity of option
T = 0.5;
% The number of points in time dimension
M = 1000;
tau = linspace(0,T,M+1);
% The length of the time interval
dtau = tau(2) - tau(1);


% Interest rate
r = 0.03;
% Dividend yield
q = 0.01;
% Volatility (Constant)
sigma = 0.2;

% alpha and beta in the discretization scheme
alpha = 0.5*sigma^2*S.^2*dtau/(dS^2);
beta = (r - q )*S*dtau/(2*dS);

% lower, main and upper diagonal of the tridiagonal matrix for the implicit
% finite difference scheme
l = -alpha + beta;
d = 1 + r*dtau + 2*alpha;
u = -(alpha + beta);

%AE = diag(d,0)+diag(l(2:end),-1)+diag(u(1:end-1),1);
% Vector to store the option prices at time k
% Option payoff at maturity
Vold = max(K - S,0);
% Vector to store the option prices at time k+1
Vnew = Vold;
for k=1:M+1
    % Dirichlet Boundary condition for European put option
    boundary = [-l(1)*K*exp(-r*(k-1)*dtau) ;zeros(N-3,1);-u(N-1)*0]; 
    % Solving the tridiagonal system for the implicit scheme
    Vnew = tridiag(d,u,l,Vold+boundary);
    % Update the option prices from k to k+1
    Vold = Vnew;
end
%%
% Interpolation to find the put price when S0 = 100 and strikes being
% 80,90,100,110 and 120.
S0 = 100; 
strike = 80:10:120;
put_fdm = strike.*interp1(S,Vold,S0./strike);
% Compute same prices with bs formulae
[~,put_bs] = blsprice(S0,strike,r,T,sigma,q);
% Plot the difference
plot(strike, put_fdm-put_bs)
title('Difference of Put prices between FDM and BS formula')
xlabel('Strike Price')
ylabel('Put Price')
```

