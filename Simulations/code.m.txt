Lx = 0.1;
Ly = 100e-6;
D = 12e-8;
k1 = 5e-6;
C0 = 5;
t_end = 1;

Nx = 200;
Ny = 50;
dx = Lx / (Nx - 1);
dy = Ly / (Ny - 1);
x = linspace(0, Lx, Nx);
y = linspace(0, Ly, Ny);
[X, Y] = meshgrid(x, y);

% Time step
dt = 0.2 * min(dx, dy)^2 / D;
Nt = round(t_end / dt);
% Velocity range (centerline velocities from 0.1 mm/s to 8 cm/s)
u_center_all = linspace(0.0001, 0.08, 8);

% Loop over each centerline velocity
for k = 1:length(u_center_all)
u0 = u_center_all(k);

% Initial condition
C = zeros(Ny, Nx);
C(:, x >= 0.01 & x <= 0.011) = C0;

% Velocity profile with slip
beta = 6 * k1 / Ly;
u = u0 / (1 + beta) * (1 - ((2 * y - Ly)/Ly).^2 + beta);
U = repmat(u(:), 1, Nx);

% Plot velocity field
figure;
h1 = contourf(X * 100, Y * 1e6, U, 100, 'LineColor', 'none');
xlabel('x (cm)');
ylabel('y (μm)');
title('Velocity Profile u(x,y) with Slip');
colorbar;
set(gca, 'YDir', 'normal');
hold on;

% Time stepping
for n = 1:Nt
Cn = C;
for j = 2:Ny-1
for i = 2:Nx-1
adv_x = -U(j, i) * (Cn(j, i) - Cn(j, i - 1)) / dx;
diff_x = D * (Cn(j,i + 1) - 2*Cn(j,i)+Cn(j, i - 1))/ dx^2;
diff_y = D * (Cn(j + 1, i) - 2 * Cn(j, i) +Cn(j-1,i))/ dy^2;
C(j, i) = Cn(j, i) + dt * (adv_x + diff_x + diff_y);
end
end

% Neumann boundary conditions (no flux)
C(1, :) = C(2, :);
C(end, :) = C(end - 1, :);
% Dirichlet boundary conditions
C(:, 1) = C(:, 2); % Inlet
C(:, end) = C(:, end - 1); % Outlet
end

% Plotting — contour view
figure;
contourf(x * 100, y * 1e6, C, 50, 'LineColor', 'none');
set(gca, 'YDir', 'normal');
xlabel('x (cm)');
ylabel('y (μm)');
title(sprintf('Solute at t = 1 s | U0 = %.1f mm/s', u0 * 1e3));
colorbar;
axis tight;

% Plot the solute "front" position
% Extract centerline (y mid) profile
center_y_idx = round(Ny/2);
figure;
plot(x * 100, C(center_y_idx, :), 'b-', 'LineWidth', 2);
xlabel('x (cm)');
ylabel('C on centerline');
title(sprintf('Centerline Solute Profile at 1s | U0 = %.1f mm/s', u0 * 1e3));
grid on;

end