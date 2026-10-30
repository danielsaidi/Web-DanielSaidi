---
title: New legal label in MKMapView
date:  2013-04-15 21:29:00 +0100
tags:  ios mapkit
icon:  swift

assets: /assets/blog/13/0415/
---

I used Google Maps in some of iOS apps, until Apple launched their own map engine. So, now I use Apple Maps. And it adds an annoying little label. Damn.

I have always admired Apple for their great design, but when it comes to `MKMapView`, which is powered by Apple Maps, I find the new legal label annoying:

![Apple Maps In Action]({{page.assets}}map.png)

See the round button in the lower-left corner? Shouldn't it be even prettier being even more in the corner? Yeah, I think so too. However, that corner is occupied by a legal label that for some reason is added to the view.

I haven't found a way to remove it with the native `MKMapView` methods, but managed to find a way to programmatically move it to the bottom-right corner instead, using a category:


```objc
#import <MapKit/MapKit.h>

@interface MKMapView (LegalLabel)

typedef enum {
    MKMapViewLegalLabelPositionBottomLeft = 0,
    MKMapViewLegalLabelPositionBottomCenter = 1,
    MKMapViewLegalLabelPositionBottomRight = 2,
} MKMapViewLegalLabelPosition;

@property (nonatomic, readonly) UILabel *legalLabel;

- (void)moveLegalLabelToPosition:(MKMapViewLegalLabelPosition)position;

@end
```

```objc
#import "MKMapView+LegalLabel.h"

@implementation MKMapView (LegalLabel)

#pragma mark - Properties

- (UILabel *)legalLabel
{
    return [self.subviews objectAtIndex:1];
}

#pragma mark - Public methods

- (void)moveLegalLabelToPosition:(MKMapViewLegalLabelPosition)position
{
    UILabel *label = self.legalLabel;
    CGPoint point = [self getPointForLabel:label atPosition:position];
    label.center = point;
}

#pragma mark - Private methods

- (CGPoint)getPointForLabel:(UILabel *)label atPosition:(MKMapViewLegalLabelPosition)position
{
    int x = 0;

    switch (position) {
        case MKMapViewLegalLabelPositionBottomLeft:
            x = label.center.x;
            break;
        case MKMapViewLegalLabelPositionBottomCenter:
            x = self.center.x;
            break;
        case MKMapViewLegalLabelPositionBottomRight:
            x = self.frame.size.width - label.center.x;
            break;
    }

    CGPoint result = CGPointMake(x, label.center.y);
    return result;
}

@end
```

I used this and was approved, so I guess you can all just use it without being concerned that you do something you're not allowed to do.