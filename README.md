# zoomble
CSS Responsive Mini-framework with Zooming sections

## CSS Responsive Mini-framework with Zooming sections

This is an experimental CSS mini-framework for building responsive sites with zoomable sections. Zoomable sections enlarge proportionally to the screen size. The system is based on relative CSS units as explained in https://css-tricks.com/building-resizeable-components-relative-css-units

I also wanted to have the font slightly zooming also if not in a zooming section. This is because on a mobile there is no much space available and people tend to keep it closer to their eyes, allowing the font to be slightly smaller.
	
For this purpose I decided to have the first breaking point at 34em (544px) and the next with interval of 14em (224px). There is also a breakpoint at 20em but it would not really necessary. It is there to put a lower limit to font size but because but we don't care much of screen smaller than 320px anyway.
	
The other breaking points are at 48em (768px), 62em (992px), 76em (1216px) and 90em (1440px). Quite standard intervals. It would be of course possible to add breakpoints anywhere but I am trying to tick on these as much as possible.
	
For the minor font size zooming I decided to go with half pixel for each section. So for the standard interval between 768px and 992px, the standard font will be 1em = 16px. For the smallest screen (<544px) it will go down to 15px (16px - 0.5px * 2)
	
All this is stored in four variables so that it can be customized
	
	$browser-context-px:        16.0px //  1em
	$breakpoints-start-px:     320.0px // 20em
	$breakopints-increment-px: 224.0px // 14em
	$font-size-increment-px:     0.5px

$browser-context-px is the ratio between px and em. In this case I am using the default 1:16
	
Both breakpoints and font sizes at each level are generate by these two functions:
	
	@function breakpointPx($level)
		// Breakpoints generator
		@return $breakpoints-start-px + $breakopints-increment-px * $level
	@function fontSizePx($level)
		// Font-sizes generator
		@return $font-size-start-px + $font-size-increment-px * $level
	
Is possible to define minimum font sizes using the function minSizeEm. For example a font that should never be smaller than 14px, also on the smallest screen size, should be
	
	@function minFontSizeEm($sizePx)
		// Minimum size of font for screen  = 320px
		@return (($sizePx / 1px) * ( 1px / emtopx(fontSizeEm(1)))) * 1em
	
That is

	14 / 15 = 0.93333em
	
Where 14 is the minimum font size we want to get and 15 is the font of the default font on the smaller screen
	
To create the zoomable sections, there is the function
	
	@function zoomSizeVw($level)
		@return ( ( fontSizeEm( $level + 1 ) * 100 ) / breakpointEm( $level ) ) * 1vw
	
So, for example, having a section with
	
	font-size: zoomSizeVw(0)
	
will make the section zoomable from 0 to infinite, proportionally at the screen size. Moreover at the screen size of 320 pixels, all fonts match their smallest size.
	
Is interesting to note that if we want to start the zoom at a different breakpoint, we need to specify in the parameter of the zoomSizeVw function because the zoom "ratio" will be different.
	
To avoid the zoomed text going too small at screen sizes < 320px is possible to have something like:
	
	.className
		@media (min-width: breakpointEm(0))
			font-size: zoomSizeVw(0)

This will be compiled as
	
	@media (min-width: 20em) {
		.className {
			font-size: 4.6875vw;
		}
	}
	
To give an upper limit to the zoom is a bit more complicated because we need to calculate the size that font reached at that point. For this calculation there is another function that return the font size reached after the zooming. The function is

	@function fixedSizeRem($level1, $level2)
		@return (((zoomSizeVw($level1) / 1vw) * (breakpointEm($level2) / 1em)) / 100) * 1rem

This function require two parameters, one for the "zoom category" because depending on from where the zoom started from, it has a different increasing ratio. The other parameter is from where ww want to match the zoom.

So for a section that we want to zoom but with a lower and upper limit we can do something like:

	.nAnnnn
		@media (min-width: breakpointEm(0))
			font-size: zoomSizeVw(0)
		@media (min-width: breakpointEm(1))
			font-size: fixedSizeRem(0, 1)

as classes name I use a serious of letter. Each letter indicate the state on each interval between breakpoints. The first letter is for the area < 320px, the last letter for the area > 1440px. The letter n means that in the area the font size is "normal", flat. A capital letter "ABCDE" means that in the area the font is zooming. The letter indicate the starting point of zooming. If a letter is duplicate it just simply means that the font keep zooming. So for example

`nAnnnnn`

means that the font is only zooming between 320px and 544px.

`nAAAAAn`

means that the font is zooming from 320px to 1440.

To have something zooming from 544px we need to use the letter B, as in:

`nnBnnnn`

There are many combination that can be used, so I generated only the necessary one.

One possible combination is this:

`nABCDEn`

In this case the font is zooming in each interval but the zooming start from 0 at the beginning of the interval. This can be used in case we have items in a grid and at each breaking point we squeeze the grid to make one more item to fit in the row.

Another option is when we want to have a zoomable section that at certain point goes from one column to two columns. 

That could look something like:

`nAACCCn`

This one will zoom until 768px and then reset the zoom to 0. It would also possible to reset the zoom to a different zoom type:

`nAABBBn`

important is not to use a zoom level before the right time. For example

`nBnnnnn`

would make font going below the minimum size because the "B" is in the position where the "A" should be.

Here there is a graphical representation of the framework:

![Graph](https://1.bp.blogspot.com/-kkVM3JynN4k/V_qkAtm3TbI/AAAAAAAAFYY/a5o_4H2qacc_tOaQ3BOB9kiGS7k2WXuawCEw/s1600/CSS%2BResponsive%2BMini-framework%2Bwith%2BZooming%2Bsections.png)
