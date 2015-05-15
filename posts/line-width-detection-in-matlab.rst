.. title: Line width detection in Matlab
.. slug: line-width-detection-in-matlab
.. date: 2015-04-28 21:23:54 UTC+02:00
.. tags: matlab, computer vision
.. category:
.. link:
.. description:
.. type: text

We wanted a simple line width estimation for a recent project. It was
meant  to analyze the width of actin bundles. After some searching for
an existing solution to the problem I only found `this paper`_.
Usually I try to use existing programs in Matlab. It is very
common for physicists to reinvent the wheel.

.. _this paper: http://www.ncbi.nlm.nih.gov/pubmed/17269626

I prepared a few images to show how the Matlab script works. I used two different line sizes for a gaussian type brush and 4 different widths for the hard edge pen tool. It should be fairly customizable to diffent approaches.

.. figure:: ../images/lineswidth/lines_size20.png
   :alt: Different strokes
   :align: center

   Differently oriented lines with brush size 20
   
.. TEASER_END

I want to check different orientations. All theses lines should be the same size.

    It should be mentioned that these drawn examples are pretty perfect
    compared to usual use cases. Commonly there would be filter applied
    to the image like gaussian blur, maybe background removal. Another
    important part is the adjustment of intensity if they differ between
    images.

The main trick of this image analysis is the `Radon transform`_. It
Creates radial projections along different angles. The same principles
are used in computer tomography to
calculate composite 3D visualizations from 2D images take from different angles.

.. _Radon transform: http://en.wikipedia.org/wiki/Radon_transform

The most prominent peaks in the Radon transform are for the angles of
the lines in an image. For that angle we look at the profile and fit a
gaussian to measure the width. That is basically all we need. The rest
of the script are a few extras to make it easier to input images.

Lets look plot of the Radon transform of the first image:

.. code:: Matlab

    function f = linewidthofimage(imagepath)
    bundleImage=imread(imagepath);

    % convert to greyscale if necessary
    if  size(bundleImage,3)>1 
        bundleImage = rgb2gray(bundleImage);
    end;

    %crop
    % if ~exist('cropRect')
    %     cropRect=[10,10,size(bundleImage,2)-20,size(bundleImage,1)-20];
    % end;
    % f=figure;
    % 
    % imshow(bundleImage,'Border','tight');
    % set(gcf, 'units','normalized','outerposition',[0 0 1 1]);
    % h=imrect(gca,cropRect)
    % cropRect = wait(h);
    % [bundleImage]=imcrop(bundleImage,cropRect);
    % close(f)

    myfigure=figure('Name',imagepath,'NumberTitle', 'off');
    bundleForPeakFinding=bundleImage;

    subplot(2,2,1); imshow(bundleImage);

    iptsetpref('ImshowAxesVisible','on')
    theta = 0:179;
    [R,xp] = radon(bundleForPeakFinding,theta);

    subplot(2,2,2); imshow(R,[],'Xdata',theta,'Ydata',xp,'InitialMagnification','fit')
    xlabel('\theta (degrees)')
    ylabel('x''')
    colormap(hot), colorbar
    axis normal  
    iptsetpref('ImshowAxesVisible','off')

    p=peakfit2d(R);
    p=round(p);

    subplot(2,2,3); imagesc(R); hold on
    subplot(2,2,3); plot(p(2),p(1),'r+')
    subplot(2,2,4); plot(xp,R(:,p(2)));
    title('R_{0^o} (x\prime)')

    [R,xp] = radon(bundleImage,p(2));
    width1=fwhm(xp,R);

    stringout=sprintf('Width:%0.2f',width1);
    disp(stringout)

    f= width1;


Two external function were used:

* `fwhm <http://www.mathworks.com/matlabcentral/fileexchange/10590-fwhm>`_
* `peakfit2d <http://www.mathworks.com/matlabcentral/fileexchange/26504-sub-sample-peak-fitting-2d/content/peakfit2d.m>`_

You can see 4 peaks corresponding to the 4 strokes (or five because one
is going over from 0 to 180 degree). One peak is right
over the borders at 0 and 179 degree. Another one is visible at 90
degree. These two are the vertical one and horizontal one.

.. figure:: ../images/lineswidth/output4lines.png
   :alt: Matlab output
   :align: center

   Matlab plot of Radon transform 

In the upper left we see the input image. In the upper right is the plotted radon transform from 0 to 179 degrees.
In the lower left, we have a little plus plotted into the most prominent peak.
This is the one that will be used. In the lower right, we have the slice of the
transform for the angle marked by the +. Now we can use the half width full maximum as a
measure of width for our *brush* size.

Now we can put everything together and write a function that takes a
path to an image file that has exactly one major line and gives the
width of it as the result.

 
.. code:: Matlab

    close all;
    clear all;
    nFilesToLookAt=0; % 0 for all
    folder_name = uigetdir;
    % Get list of all *.png files in this directory
    % DIR returns as a structure array.  You will need to use () and . to get
    % the file names.
    imagefiles = dir(fullfile(folder_name,'/*.png'));      
    nfiles = length(imagefiles);   
    if nFilesToLookAt==0
        nFilesToLookAt=nfiles;
    end

    AllWidths=[];
    for i=1:round(nfiles/nFilesToLookAt):nfiles
        currentfilename = fullfile(folder_name,imagefiles(i).name);
        w=linewidthofimages(currentfilename)
        AllWidths=[AllWidths w];
    end

    figure;
    plot(AllWidths);

    disp(sprintf('mean width:%0.2f',mean(AllWidths)));
    disp(sprintf('median width:%0.2f',median(AllWidths)));

This procedure queries a folder name and calculates the
widths of all images inside. Afterwards it plots the resulting widths.
I created a folder with single images of these lines:

.. figure:: ../images/lineswidth/lines_pen.png
   :alt: Matlab output
   :align: center

   different sizes

And this was the output:

.. figure:: ../images/lineswidth/pens.png
   :alt: Matlab output
   :align: center

   different sizes
           
This can be useful if you have a image series with timestamps
and want to see if the object get wider.

And that is all to it. Possible changes include opening a GUI to
specifically select a line (currently commented out). One can also
detect several peaks at one which is provided with the used peak
finding function.
