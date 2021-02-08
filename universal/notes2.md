## june 8

* frbcat.org
* FRB 180814.J0422+73
* FRB121102

* Andersen 2019: "repeater bursts are generally wider than those of CHIME/FRB bursts
that have not repeated, suggesting different emission mechanisms."
	* should get some non-repeaters as well as repeaters

* CHIME data is not available publicly.
    * CHIME data IS available publicly! https://chime-frb-open-data.github.io/

* ASKAP Data 50 FRBs:
    * https://data.csiro.au/collections/#collection/CIcsiro:34437v3
* Bursts with data on frbcat.org:
    * see `frbcat_frbs_with_data.txt`

## june 9
* downloaded data for FRB180110 from ASKAP site
    * format in SIGPROC filterbank format (http://sigproc.sourceforge.net/sigproc.pdf), http://sigproc.sourceforge.net/
    * PRESTO (https://www.cv.nrao.edu/~sransom/presto/) is related
* downloaded FRB180814 (CHIME)
    * who is making these shitty archaic data repositories? Their jnlp download manager downloads empty files and I'll have to write a wget script if I don't want to click a million links

* CHIME cfod package seems to work fine... but doesn't look like a burst

## june 10
* cant seem to see anything in the 180814 data (both of them)

## june 13
* tried compiling SIGPROC, its filled with fortran errors. maybe I'm using a different version of fortran. they seem to use F77
	* potential alternate? https://github.com/SixByNine/sigproc
	* try PRESTO (https://www.cv.nrao.edu/~sransom/presto/) instead? which handles SIGPROC files
	* tried PSRSoft, which is like a package manager for pulsar shit but sixproc build fails http://www.pulsarastronomy.net/wiki/Software/PSRSoft

## june 15
* Found another secret CHIME/FRB repo https://github.com/CHIMEFRB
* compiling Mike Keith's version worked immediately lol (https://github.com/SixByNine/sigproc)

## june 16
* askap: dont do `reader *.fil > data.csv` , it only takes the last beam file
* `splice` seems to just append columns to the data
* askap frb180110 is in beam 31, 32. DM (from shannon et al.) is 715.7 pc/cm^3
* to dedisperse example:
 `dedisperse 2018-01-10-07:07:52_0000000000000000.000000.31.fil -d 715.7 -b 336 > ddbeam31.fil`
 `reader ddbeam31.fil > dd_beam31.csv`

## june 17
* dedispersion can be a little involved, but at its core you use the DM to compute the time delay then shift the frequency observations by that time delay. See sigproc documentation, Lorimer et al. 2007, Amiri et al. 2019
* I was curious if you could get good correlations from png images of the bursts, and seems like you can, so long as you remove the noise so that there's enough snr between the burst pixels and the noise pixels. This method feels very sketchy, but maybe if you make sure the pixel grid corresponds to the data grid (ie. your image shouldn't be higher resolution than the png of the data) then you can extract correct physical parameters of the burst
* A safer route is to try PRESTO (https://www.cv.nrao.edu/~sransom/presto/), which has a dedispersion planner thing that might be good. Otherwise we can write our own dedispersion thing

## june 23
Repeater Sources: See repeaters.csv
Prioritize 171019, 180916.J0158+65, and 180814.J0422+73

## june 24
* new data from chime is in a better format: https://chime-frb-open-data.github.io/
    * data doesnt match paper

## june 26
* i downloaded the pdf figure from the arxiv source for the 180916 paper, and turns out you can open it in photoshop and extract the images automatically with high fidelity! no screenshotting, no interpolation, pixel scale is an integer, very clean way to get the data from the figures. I now have 25 images that can be used for each burst

## june 29
* rgb2gray source: https://stackoverflow.com/questions/12201577/how-can-i-convert-an-rgb-image-into-grayscale-in-python
* bursts 31, 32 show a burst in the same band with obviously different sub-burst drifts
* first attempt yields 12/25 good fits for 180916+J0158+65. Can probably get a few more
* y pixel scale is variable.. seems like plus or minus 1 pixel

## june 30
* used pixel scale to place physical axes on the images. save fit parameters so its faster

## july 1
* when finding slope from the fit angle we correct for the pixel scale as
$$
\tan\theta	=\frac{y}{x}
\frac{y/y_{s}}{x/x_{s}}	=\frac{y}{x}\frac{x_{s}}{y_{s}}=\frac{x_{s}}{y_{s}}\tan\theta
$$

* sigmax and sigmay are in pixel space, so they must be scaled by the pixel scale

* early plot suggests that the bursts from 180916+J0158+6 are on trend with the bursts from 121102
* ~need to investigate why the t_w for bursts 19 and 21 are negative~ just make sure sigmax and sigmay are positive

## july 2
* need to investigate differences in tw and how they correspond to the figures
* Ziggy responded to my github issue, downsampling properly now based on his suggestion. I had tried decimating the data to 'downsample' but that didn't work so I downsample via averaging now, which is what I should have tried in the first place
* downsampling produces some streaks in the burst. Will need to resolve this eventually

## july 3
* do boxcar downsampling for chime bursts.. convolve and sample (?)

## july 4
* did errors for chime 180916 bursts

## july 6
* in FRB2020 talks
    * http://chime-frb.ca/repeaters
* frb171019 is in beams 20, 21, 27, 28
* there's an implementation of dedispersion here: https://github.com/danielemichilli/DM_phase/blob/master/DM_phase.py#L553
    * the key line is `shift = (k_DM * DM * (reference_frequency**-2 - freq**-2) / dt).round().astype(int)`
* The published DM doesn't give me what is shown in Kumar et al. 2019, i have to use a larger value, like the other burst. Not sure why

## july 7
* add 180916 bursts to 121102 bursts
* Tried to use DM_phase to get a DM for 171019 but eeets not woooorkinggggg. Best value it gives is 380, which is worse than what I could do by eye
    * ok it works if you use a large range and fine grid kinda

## july 8
* read hessels
* do 180814
* no matter the DM the relationship is just shifted (duration order is preserved)

## july 11
* shri from FRB2020 gave a good talk about DM
* different DM's preserve "steepness" order but I don't think it will affect durations. So different DMs should only affect which A/tw curve the points land on (by only affecting A)
* kinda fix my banding problem by using np.nanmean instead of np.mean when downsampling
* 180814.J0442+73 data looks like shit, no bursts, noisy...

## july 15-16
* opened another chime issue, gonna process the 180916 data properly in the meantime (ie. not from images)

## july 17
* i found fits using the proper 180916 data and it mostly finds the same fits as the image technique did

## july 20
* calculate drifts wip

## july 23
* ive calculated the drifts from the data itself and it very closely matches what i got from the images, which validates that method. sometimes the quick and dirty way is also the correct way.

## july 25
* using parameter guesses to get better fits on some bursts
    * tried a shotgun blast (every fit starts the search with the same ellipse. i chose the ellipse for burst 14 cuz it looked reasonable) --> doesn't work
    * can fit some bursts when i feed it a good initial start as well as bounds on the amplitude.
    * I noticed the initial guess from moments() is often very bad which might be why some fits are not found
    * clipping the autocorrelation also works
* finalized 180916 fits and drifts, 16 points in total

## july 26
* stacked and dedispered the 180814 bursts, but having trouble finding them in the 16 second cutouts.
    * needed to use the ubuntu subsystem to get around the memory error thing

## july 27
* finalized 180916 and updated the figures, just 180814 is left

## july 28
* fit for 180916:15 can be tweaked a little. its too wide for the autocorrelation it seems

## aug 3
* did 180814 via images
    * pixelxscale is ~348px/129units, y is ~650/64units
* there is an inconsistency in burst181028: i get a different duration than the chime paper

* I was accidentally plotting the raw drift instead of the drift corrected to the Michilli frequency... The fit is MUCH BETTER!! All three sources fall very close to the fit

## aug 5
* plotting drift/nu_obs as chris suggested (to isolate the fit parameter to properties of the source) still shows good fit, and separation between sources that shows the fit is valid over distinct regions of the domain
* redshifts:
    frb121102: z = 0.19273 (josephy et al. 2019, frbcat.org)
    frb180814.J0422+73: z < 0.1 (amiri et al. 2019)
    frb1801916.J0158+65: z = 0.0337 (chime 2020b et al.)
* added redshift corrected plots

## aug 6
* fix redshift corrected plots, redshift makes slight difference

## aug 7
* added extra dedispersion to 180916 data to see how it affects the trend
    * there is a definite shift in the data, however, as we argued in our referee report, i think the existence of a relationship is still there regardless of the DM. Now I was a bit limited in the DMs I could pick because of the width of the data set (which I could have worked around by zero-padding) but more importantly because the bursts would very quickly become positive outside of this delta DM range. I think this is a consequence of how the DM is found: it is an optimization process, so moving away from the found DM in either direction by any significant steps will be obviously incorrect. This is in fact a good thing: it means the DMs that are published are not as terrible as we've been assuming, even when the burst drifts come out slightly positive like in the case of frb180814: the DM can certainly be tweaked a little bit to get a negative slope, but any significant changes to the DM can be safely ruled out.
    * I think this means that (a) having the right DM is important in obtaining a good fit between sources and (b) the fact that we have a good fit isn't coincidence, but a consequence of well-selected DMs on the part of CHIME and other authors. I want to therefore cautiously suggest that, if indeed there is a shared fit between sources, you can dedisperse bursts from other repeaters by picking a DM that lines that source's bursts with our fit

## aug 8
* tried splitting onto 100ms windows but the 180814 burst still doesn't show up. Maybe the dispersion is bad? Maybe I'll try the other bursts? will need to remove the noise for those bursts..

## aug 14
* Ziggy replied with some suggestions about checking the true error of the drift measurement, and said he'll help with 180814 eventually. He also pointed out that the DM for repeaters changes.. We need to understand how this affects the trend and probably do a proper error analysis for the paper
* FRB121102 DM Ranges
    * In the data we've used:
        * FRB121102 Gajjar (11A and 11D): 565 pc/cm^3,
        * FRB121102 Michilli            : 559.7 pc/cm^3
        * FRB180916.J0158+65            : 348.82 pc/cm^3
        * FRB180814.J0455+73            : 189.4 pc/cm^3
    * FRB121102 Spitler 2014 553-569 pc/cm^3
    * FRB121102 Gajjar 2018 562-636 pc/cm^3 (583-601 pc/cm^3 for the bursts we've used)
    * FRB121102 Michilli 2018 <= 1% of 559.7 = approx 6 pc/cm^3
    * FRB180916 CHIME 2020: 348-350
    * FRB180814.J0455+74 Amiri 2019 188.9-190 pc/cm^3


## aug 15
* If I had read gajjar et al. 2018 more carefully I would have seen packages and data releases for all their bursts:
    * http://seti.berkeley.edu/frb121102/technical.html
    * https://github.com/mtlam/PyPulse
    * perhaps we can add more 121102 bursts to our plot
* Something we should say "we choose the structure-optimizing DM over the S/N-maximizing DM whenever possible"
* There are several orders of magnitude between the drift rates of pulsar pulses and FRBs (e.g. Hewish 1980)
* In order to incorporate the error on the DM into the drift measurement, find what the drift is at each end of DM range? But this doesn't take into account the variation of DM between bursts..
* DM can also be frequency dependant (Lam et al. 2020) (at least based on pulsar observations, but DM is a value related to the ISM not the source so presumable this can also be true for FRBs)
* A careful discussion of the DM and its influence on the validity of our result will be essential for convincing radio astronomers
* I should be able to relate the delta t from the dedispersion equation to the observed slope. With that, I can just calculate directly what the drift error should be from the DM error

## aug 17
* paper notes:
    * main text should include brief discussion of DM if possible, but the full argument can live in the "Materials and Methods"
    * I can write the entier supplementary info section which includes the Materials and Methods
        * I'd like to include every burst and autocorrelation if possible, labeled with the inferred drift measurement
        * a description of the error analysis (esp. the crazy equation for tw) should be given
        * Essentially summarize the document that processes the drifts and creates the trend figure; describe how the trend is found as well
        * Detail how we control for DM
* The heart of our problem is this:
    * We are telling people "if you dedisperse all the bursts from a repeater source to the same DM then you will see this trend between the drift rate and duration.", which is a true statement.
    * However, the first question is "Why should I dedisperse to the same DM given that the DM can vary per burst, and sometimes, like in FRB121102, vary in time?"
    * What is our answer to this? We can say that for some sources (like 180916 and 180814) the range of DMs is very narrow. We can also say it is reasonable to assume that the DM is a property of the medium and not the source.

## aug 19
* met with fereshte and chris: proceed with plan regarding error bars

## aug 20
* wip on refactoring for multi-dm error analysis

## aug 27
* got data for dm vs. drift for frb121102 with ten trial DMs within the error range cited by Michilli.
* As a way of making our paper more convincing Martin suggested plotting angle vs duration instead of drift vs duration
    * this has the advantage of avoiding the discontinuity the drift experiences
    * shifting the DM on the angle vs duration plot will show that the DM just shifts the plot
    * I will also plot drift vs. DM and angle vs. DM
* drift range is huge and arbitrary due to discontinuity. get the range of angles, then use that as the angle error and calculate the drift error from that

## aug 29
* for 180916 plot angle vs duration and angle vs trial dm. you already have the data for this

## sept 4
* 1. get relationship of angle to tw from model, and add fits to angle plot
    1.2 how does tw change
* 2. start writing methods
* 3. process 180814 asap

## sept 7
* burst 32 has an extremely long burst duration and small angle for ddm -1/2
    * the solver paramters make no sense (negative amplitude, etc.) so either exclude it or fix it
* the angle vs tw model is -atan(A/tw)

## sept 9
* the angle vs tw model is actually atan(tw/A)

## sept 11
* lol i missed a lot
    * fix the points on angle vs tw that have bad fits
    * check martin's deldta dm derivation and figure out delta theta
    * chris needs data for his idea
    * check martin's determination of beta nu_0 stuff
* ~burst 32 is still fucky on ddm = -0.5~ fixed

## sept 14
* i have delta_DM, i need to know what delta theta is given that (which is presumably what Martin derived)
* found offset fits for angle vs duration
* wip on calculating deltatheta.. need to read the doc more

## sept 15
* chris reproduced my results and showed that fit is optimal at a certain DM
* my dedispersion has an extra minus sign so when I dedispersed to -1/2, 1, and 2 pc/cm^3 what I actually did was dedisperse to 1/2, -1, and -2 pc/cm^3
    * fixed

## sept 18
* 180914 bursts are overdispersed

## sept 19
* the angle vs tw plot shows that even within a narrow range of dms, you can dedisperse bursts to validate any model you want

## sept 21
* generated the parameter files for a range of DMs for FRB180814
* generated plots. the three bursts fit the trend

## sept 23
* see meeting notes

## sept 25
* If I use the proper resolution for 180917 i can no longer see the burst
* when dispersed, the chime bursts seem to bow the other way than say the lorimer burst.. why?

## sept 28
* there was a mistake in my dedispersion code for the 180916 trial dm points: they bend the right way now.
* Martin wrote up a document that explains how to handle differing reference frequencies
* started writing methods
    * Science guidelines: https://www.sciencemag.org/authors/instructions-preparing-initial-manuscript
* Amiri et al. 2019 has a model for the sub-bursts of 180917, maybe I should use it when splitting up the bursts

## sept 29
* i need to redo the angle vs duration plot for 180916 with the proper dedispersion code

## sept 30
* Remaining analysis:
    * 180916 angle vs duration
        * add padding when dedispersing to handle roll overs
    * split up and process burst 180917 from frb180914
    * find center_f properly for CHIME data

## oct 1
* first draft of methods is done
* found center_f. some of them look weird but they're mostly good

## oct 5
* no need to zero pad, just find the max peak and roll the entire array by the fixed amount required to center the burst
* redid the analysis for FRB180916 with corrected dedispersion. No major change. Fit improved slightly. DM variations still act as a rotation of the autocorrelation.
    * notably center_f didn't change results much either

## oct 7
* split up 180917 and found fits. need to center the view for trial DMs
    * i think CHIME made a mistake dedispersing 180917 but we'll see how the points fall.

## oct 12 - oct 16
* paper todos:
    * stampcards
    * describe trial dm plot (for FRB180916)
    * understand trial dm plot for FRB180914, and add fits to those plots
    * describe burst exclusions
    * edit
* more todos:
    * you should scale sigma by root(red_chisq) not red_chisq: https://en.wikipedia.org/wiki/Weighted_arithmetic_mean#Correcting_for_over-_or_under-dispersion
    * martin's plot

## oct 13/14
* center_f wasn't being read into the shared law plot, fixed it. Fit is better now. the more i fix the better the sources match
* added paragraph about trial dms to paper
* made martin's plot for drift vs nu_obs
* still have some todos from the above list

## oct 14
* more more todos:
    * simplify text with diagram of ellipse defining the terms
    * consider equal axes for stampcard
    * write the dm selection stuff, esp for FRB180814
    * error bars on angle
* 180917 bursts have chisq << 1, which is supposed to indicate overfitting?

## oct 22-23
* submission todos:
    * angle error bars    : done
    * stampcard           :
    * fix chisq error bar : done
    * burst exclusion     : done
    * drift vs frequency  :
    * maybe simplification diagram?
* reasons for exclusion:
    * burst smearing
    * low snr
    * top of band/bottom of band
    * noise distorts shape
* stampcard
    * 1. 121102 (23x2)
        * does this even exist
    * 2. 180916 (16x2)
        * some of the centers are fucked
    * 3. 180814 (6x2)

## oct 26
* nuobs matters on angle vs duration plot, for after submission
* gajjar center f
* "Correlation 15" use 15A
* contour color
* replot fig 1 after center_fs
* fig 1 parameter uncertainties

* submitted.

## oct 29
* verify error bars and error equations

## nov 17
* the equation on tw was correct, the error equation had an error. they are now so small they are invisible

## jan 13
* start work on ref response
* ProcessBursts.ipynb (frb121102) is unwieldy and should probably be re-written..
* S/N optimized DM gives the wrong DM, since it stacks components to increase the SNR. Structure optimized DM is the only valid measure of DM for complex bursts.
* generated differently dispersed data for the michilli 121102 bursts

## jan 14

* there's a lot of moving parts so let me summarize the ref response with a single sentence:
    * "How have you studied the fact that the measurements of drift and burst duration, and thus the relationship between them, depends on the choice of DM and the way bursts are split?"
        * DM choices can be made in terms of individual bursts or in terms of the source
        * For individual bursts, DM can be chosen by maximizing S/N, or by maximizing the structure parameter. Not all bursts have enough S/N or components to by dedispersed by structure. The DM found by S/N varies significantly (is always greater?) than the structure-optimized DM for bursts with multi-components.
        * From the individually found DMs, one can make choices about the DM by considering the time dependance of the DM for the source, the statistical variation of the DMs found (e.g. take an average DM), and/or on which basis the DM was found.

## jan 21
* add plotVsDuration in driflaw.py, working well,
    * add annotations and exclusions next
* compute the min and max drift per burst using the range of DMs

## jan 22
* annotate series with a list of indices that correspond to the frames list
* exclude errant points on the user side, at least for obvious issues like positive drift.
    * exclude other points (like points that represent multiple bursts before splitting) with an exclusions array and df.drop()
    * reason for excluding points:
        * positive drift, we assume they are non-physical and an artifact of over dedispersion
        * infinite drift/ie. huge error bars. burst is too vertical. this the same class exclusion for positive drifts: unphysical
            * exclude with a filter on the drift error. too large error means you're out. this clearly indicates a poor measurement
* use `%matplotlib qt` to get the interactive window and explore errant points directly.
* the frb121102 data has angle problems
    * it didn't have the absolute value when comparing sigmax and sigmay. consolidating library code will eventually fix these bugs
* even though 1/drift allows you to eliminate infinity, those measurements of drift that are close to vertical will still have huge error
    * the measurement of drift angle is superior to drift in every way, especially in the context of DM variations
        * the measurement of drift angle and its error varies uniformly as you vary the DM... but drift never will
        * however if we believe positive drifts to be unphysical then we have to exclude angles at and close to pi/2. In this case the size of the drift error can be used as a filter by requiring a certain standard of it.
* if you exclude a point because its drift has a huge error at a certain DM, you should exclude it from all other DM trials as well.
    * A huge drift error at a trial DM in a range of DMs means the drift for that burst is unconstrained, and meaningless, in that entire range of DMs.

## jan 26
* todo:
    * frb121102 variation data is off compared to fig 1, some most dms are not being plotted? >> center_f was in ghz instead of mhz
    * compute error range from dm >> see driftranges() function. need to integrate it with the rest of the library
    * percent error exclusion logic

## feb 1
* The t_w error bars on the data is different slightly. Run the old notebook and ensure they're consistent.
    >> the figure in the arxiv submission is wrong, we fixed its t_w error bars in the nov 17 and dec 7 commits
* duration appears to be wildly unconstrained across the dm range
    >> the ranges I computed are physical values, not errors. So I can't just pass them as errors
* shaded region plotting: https://stackoverflow.com/questions/12957582/plot-yerr-xerr-as-shaded-region-rather-than-error-bars
* The ranges we computed are not stastical errors, so why are you using them?
    * The DMs quoted for each burst have a distribution, however this distribution does not capture the range of DMs for all the bursts taken together. So we can't use something like Ziggy's dfdt to measure the drift covariance because the quoted DM errors do not take into account the DMs distribution for the whole source. That is, the actual distribution of possible DMs is much larger than any of the DM errors found on an individual burst. We could try to characterise the distribution of DMs by looking at all of a source's bursts, but you would still need to exclude bursts based on non-physical drifts. So it's simpler and more conservative to just compute a range of drifts given a range of DMs and accept all reasonable drifts as "possible", even if some of those measurements are unlikely given an underlying DM distribution (which is unknown). This means that our range will in fact be larger than true error bars on the drift measurements. We can do this because we don't actually really care what the actual linear drift rate is, we just need to understand its trend relative to other bursts from the same source. So even though we cannot measure the linear drift rates precisely, we can still learn about the relationship between them, so long as we are careful to exclude unconstrained linear drift rates (drift rates that go to infinity).
* The underlying time precision on 180916's bursts is not very good so those points have large ranges.
* Now that the plotting pipe is working, I need to study what DM ranges are appropriate, and fit our model given the new ranges.

## feb 2
* see aug 14 note for DM ranges. im fine with what we have for now
* I don't think its a good idea to fit in a log-log space. Do it in linear.
    >> looks like a worse fit on log scale, errors are still unreasonably small
* I don't get the errors on ODR. Fit on every DM frame and make a range of fit parameters just like you made a range of drifts.
    >> done. looks good. But did not characterize which were good fits. Which I guess doesn't matter, since it would make the region narrower if I did.
* make sure you label all error bars and shaded regions clearly since I'm not using them in the standard way
* wait can i just take the list of A parameters and naively compute its standard deviation? is that dumb?
* see what happens if you exclude by burst id and not just by value. That is, if one burst is bad at a certain dm, drop that burst at all dms
* add the gajjar and chime 121102 bursts

## feb 4
* refactoring the Gajjar bursts
    * automated burst separation is possible via a derivative of the time series, but to be honest at trial DMs there's no guarantee that they aren't on top of each other. So it's best to separate them at a single DM, pad them with  background noise, and then dedisperse them to different DMs.
    * Actually the structure limits the DM possible for these bursts. Any DM where they are overlapping should be excluded. So maybe instead use the time series to count the number of distinct bursts and if it isn't what's expected (4 in the case of 11A) then rule out that DM. Hmm...
    * This means that pulse trains can be used to greatly constrain the possible DM, more so than multiple individual bursts. With a tightly constrained DM from pulse trains from multiple sources, you can tightly constrain the relationship between the sources. A version of the drift vs duration plot that ONLY has pulse trains on it should be very informative.
* omg that x/y vs y/x fitmap bug is back NOOOOOOOOO

## feb 7
* the gajjar spectra have many more frequency channels than time channels. fit results improve when you subband the frequency channels
    * could just be due to the increased snr
* i think the x/y bug was really just a difficulty fitting bug. feeding curve_fit with better initial guesses makes good fits.
