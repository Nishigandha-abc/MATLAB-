1) 
clc;
clear;

N = 800; % Number of symbols

% Seed the random number generator (Octave version)
randn("state", 0);

% Generate random binary sequence
am = 2 * (rand(1, N) > 0.5) - 1; % 1 and -1

% Manually add white Gaussian noise to the sequence
SNR = 20; % Signal to noise ratio in dB
noisePower = 10^(-SNR / 10); % Convert SNR to noise power
noise = sqrt(noisePower) * randn(1, N); % Generate Gaussian noise
am1 = am + noise; % Add noise to the signal

fs = 5; % Sampling frequency in Hz

% Define the sinc filter
sincNum = sin(pi * [-fs:1/fs:fs]); % Numerator of the sinc function
sincDen = (pi * [-fs:1/fs:fs]);    % Denominator of the sinc function

% Avoid division by zero
sincDenZero = find(abs(sincDen) < 10^-10);
sincOp = sincNum ./ sincDen;

% Set values at zeros to 1 (sin(pi*x)/x = 1 for x = 0)
sincOp(sincDenZero) = 1;

% Raised cosine filter parameters
alpha = 1.0; % Roll-off factor
cosNum = cos(alpha * pi * [-fs:1/fs:fs]);
cosDen = (1 - (2 * alpha * (-fs:1/fs:fs)).^2);

% Avoid division by zero in cosine filter
cosDenZero = find(abs(cosDen) < 10^-10);
cosOp = cosNum ./ cosDen;
cosOp(cosDenZero) = pi / 4;

% Band-limiting raised cosine low-pass filter
gt_alpha1 = sincOp .* cosOp; % Band-limiting RC lowpass filter

% Upsampling the transmit sequence
amUpSampled = [am1; zeros(fs-1, length(am))];
amU = amUpSampled(:).';

% Filtered sequence
st_alpha1 = conv(amU, gt_alpha1, 'same'); % Apply convolution
st_alpha1 = st_alpha1(1:4000); % Take the first 4000 samples

% Reshape for plotting
st_alpha1_reshape = reshape(st_alpha1, fs*2, N*fs/10);

% Plot the results
subplot(2,1,1);
plot([0:1/fs:1.99], real(st_alpha1_reshape).', 'b');
title('Eye Diagram with \alpha = 1.0');
xlabel('Time (s)');
ylabel('Amplitude');
axis([0 1.99 -1.5 1.5]);
grid on;
