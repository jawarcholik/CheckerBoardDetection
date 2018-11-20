clear variables
close all
% Open movie file.
movieObj = VideoReader('checkers.mp4');
nFrames = movieObj.NumberOfFrames;
fprintf('Opening movie file with %d images\n', nFrames);
% Go through movie.  We don't need to process every frame.
for iFrame=1:10:nFrames
    I = read(movieObj,iFrame);
    fprintf('Frame %d\n', iFrame);
    % Reduce image size; is faster and we don't need full size to find board.
    if size(I,2)>640
        I = imresize(I, 640/size(I,2));
    end
    figure(1), imshow(I), title(sprintf('Frame %d', iFrame));
    hold on;
        % Find the checkerboard.  Return the four outer corners as a 4x2 array,
        % in the form [ [x1,y1]; [x2,y2]; ... ].
        [corners, nMatches, avgErr] = findCheckerBoard(I);
%         figure,imshow(I);
       
        line([corners(1,1),corners(2,1)],[corners(1,2),corners(2,2)]);
        line([corners(2,1),corners(3,1)],[corners(2,2),corners(3,2)]);
        line([corners(3,1),corners(4,1)],[corners(3,2),corners(4,2)]);
        line([corners(4,1),corners(1,1)],[corners(4,2),corners(1,2)]);
        
        Pts = [0,0; 600,0; 600,600; 0,600];
        
        ref2Doutput = imref2d( ...
            [600, 600],...% Size of output image (rows, cols)
            [0, 600],...% xWorldLimits
            [0, 600]);           
        
        Tform = fitgeotrans(corners, Pts, 'projective');
        I1 = imwarp(I, Tform, 'OutputView', ref2Doutput);
        I2 = rgb2gray(I1);
        
%         I1 = imcropI1, [corners(1), corners(2), corners(3), corners(4)]);
        figure, imshow(I1);
        [centers, radii, metrics] = imfindcircles(I2, [10 30], 'Sensitivity', .85, 'EdgeThreshold', .05);
%         viscircles(centers, radii, 'EdgeColor', 'b');
        for i=1:length(centers)
            if (getIntensityDifference(I2, round(centers(i,1)), round(centers(i,2)), round(radii(i))) < .05)
                % Draw Blue(Black) Circle
                viscircles(centers(i,:), radii(i), 'EdgeColor', 'b');
            else
                % Draw White Circle
                viscircles(centers(i,:), radii(i), 'EdgeColor', 'w');
            end
        end

        pause;
end


function diff = getIntensityDifference(I, x0, y0, R0)
% This function computes the average intensity difference between the
% pixels just inside a circle of radius R0, and the pixels just 
% outside the circle.
% Input parameters:
%   I: the input grayscale image
%   x0,y0:  the center of the circle
%   R0:  the radius of the circle
% Output:
%   diff: the average intensity difference
 
% Get the x,y coordinates of all pixels near this circle.
dR = 4;
xMin = max(x0-R0-2*dR, 1);
xMax = min(x0+R0+2*dR, size(I,2));
yMin = max(y0-R0-2*dR, 1);
yMax = min(y0+R0+2*dR, size(I,1));
[X,Y] = meshgrid(xMin:xMax, yMin:yMax);
R = ((X-x0).^2 + (Y-y0).^2) .^ 0.5;
 
% Flag those that are less than distance R0-dR from the center, but
% more than R0-2*dR from the center.
Rinside = (R < (R0-dR)) & (R > (R0-2*dR));
 
% Get (x,y) coordinates of the inside pixels.
Xinside = X(Rinside);
Yinside = Y(Rinside);
 
% Get equivalent linear indices.
indices = sub2ind(size(I), Yinside, Xinside);
 
% Get average intensity of those inside pixels.
intensityInside = mean(I(indices));
 
% Flag those that are more than distance R0+dR from center, but less
% than R0+2*dR from center.
Routside = (R > (R0+dR)) & (R < (R0+2*dR));
 
% Get (x,y) coordinates of the outside pixels.
Xoutside = X(Routside);
Youtside = Y(Routside);
 
% Get equivalent linear indices.
indices = sub2ind(size(I), Youtside, Xoutside);
 
% Get average intensity of those outside pixels.
intensityOutside = median(I(indices));
 
diff = intensityInside - intensityOutside;
end