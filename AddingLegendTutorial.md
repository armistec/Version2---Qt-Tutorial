## Qt QML Tutorial

### Creating a Legend based on available sublayers in ESRI App Studio Qt

##### This tutorial begins assuming the user has an ArcGIS online user account and has already downloaded ESRI AppStudio for Desktop
##### Relies on ESRI's example for adding a legend dependant on features.

#### More examples can be found on the [ESRI AppStudio Github](https://github.com/Esri/arcgis-appstudio-samples "ESRI Samples")

##### First, the modules needed for the application are imported

```QML
import QtQuick 2.7
import QtQuick.Layouts 1.1
import QtQuick.Controls 2.1
import QtQuick.Controls.Material 2.1
import QtGraphicalEffects 1.0
import ArcGIS.AppFramework 1.0
import ArcGIS.AppFramework.Controls 1.0
import Esri.ArcGISRuntime 100.1
import "controls" as Controls
```

##### Then, we create the basic structure of the application
##### The first section of code determines the intial dimentions of the applcation, then scales them appropriately, then the page properties are set for the overall map structure. The ```Material.backgound``` tag allows a user to change the background color that displays upon loading of the application

```QML
App {
    id: app
    width: 414
    height: 736
    function units(value) {
        return AppFramework.displayScaleFactor * value
    }
    property real scaleFactor: AppFramework.displayScaleFactor
    property int baseFontSize : app.info.propertyValue("baseFontSize", 15 * scaleFactor) + (isSmallScreen ? 0 : 3)
    property bool isSmallScreen: (width || height) < units(400)

    Page{
        anchors.fill: parent
        header: ToolBar{
            id:header
            width: parent.width
            height: 50 * scaleFactor
            Material.background: "#8f499c"
            Controls.HeaderBar{}
        }
```

##### The next sections all fall within the ```App``` component 

##### We are breaking down the components of the ```content item``` that ultimately go into creating a map with multiple features and a legend that shows the categories of all sub layers

##### Here the ```mapView``` component creates the base framework where we will add layers

```QML
contentItem: Rectangle{
            anchors.top:header.bottom
            MapView{
                id:mapView
                anchors.fill: parent
```

##### This next bit of code is housed as a child of mapVIew. This may look like a large block - but this is where layers are added and the inital view of the map is set. Comments within the code provide direction on ways to customize.

```QML

// Again, this will be a child under mapView
                Map {
                    id: map
                    // This creates the call to automatically search added layers for legend attribute info
                    autoFetchLegendInfos: true

                    // Choose the Basemap type, adding it to the Map
                    BasemapTopographic {}

                    // Add a tiled layer as an operational layer - these can be layers that are already hosted in an acrGIS online account, the easiest way to pull in your own features, as interaction or access to a server is not needed
                    // This is a sample ESRI layer of a soil survery tiled map. Using a tiled layer also helps speed up the draw time for the layer
                    ArcGISTiledLayer {
                        id: tiledLayer
                        url: "http://services.arcgisonline.com/ArcGIS/rest/services/Specialty/Soil_Survey_Map/MapServer"
                    }
                    
                    // Add a feature layer as an operational layer - this is an additional option to load rest services, this can be particularly useful if data is being entered or updated through mobile apps or ArcGIS desktop to a rest service, because then changes will automatically be pushed to your app without needed to update ArcGIS online layers
                    // Again, this is a sample layer from ESRI
                    FeatureLayer {
                        id: featureLayer
                        featureTable: ServiceFeatureTable {
                            url: "http://sampleserver6.arcgisonline.com/arcgis/rest/services/Recreation/FeatureServer/0"
                        }
                    }

                    //  Finally the initial view of the app (when the screen first opens) is set here
                    //  The format here may look unfamilar, 
                    initialViewpoint: ViewpointCenter {
                        center: Point {
                            x: -11e6
                            y: 6e6
                            spatialReference: SpatialReference {wkid: 102100}
                        }
                        targetScale: 9e7
                    }
                }
```

##### Next, adding a signal to the user that layers are being added - helpful user feedback

```QML
// Busy Indicator
                BusyIndicator {
                    id:mapDrawingWindow
                    anchors.centerIn: parent
                    height: 48 * scaleFactor
                    width: height
                    running: true
                    Material.accent:"#8f499c"
                    visible: (mapView.drawStatus === Enums.DrawStatusInProgress)
                }
```

##### Now that we have the map layers added, it is time to add the legend, first the properties for the container are defined here

```QML
// Create outter rectangle for the legend
                Rectangle {
                    id: legendRect
                    anchors {
                        margins: 10 * scaleFactor
                        left: parent.left
                        top: mapView.top
                    }
                    property bool expanded: true
                    height: app.height/2 - 30 * scaleFactor
                    width: 175 * scaleFactor
                    color: "lightgrey"
                    opacity: 0.95
                    radius: 10
                    clip: true
                    border {
                        color: "darkgrey"
                        width: 1
                    }
```

##### Now adding animation to open or close the lenged

```QML
// Animate the expand and collapse of the legend
                    Behavior on height {
                        SpringAnimation {
                            spring: 3
                            damping: .8
                        }
                    }

                    // Catch mouse signals so they don't propagate to the map
                    MouseArea {
                        anchors.fill: parent
                        onClicked: mouse.accepted = true
                        onWheel: wheel.accepted = true
                    }
```

##### Finally bringing it all together to add the styling to the lenged window, this is pulling attributes from each layer file added above. This code will run without additional edits, but changes can be made to change how the legend looks, the placement of text and icons

```QML
// Create a list view to display the legend
                        ListView {
                            id: legendListView
                            anchors.margins: 10 * scaleFactor
                            width: 165 * scaleFactor
                            height: app.height/2 - 55 * scaleFactor
                            clip: true
                            model: map.legendInfos

                            // Create delegate to display the name with an image
                            delegate: Item {
                                width: parent.width
                                height: 35 * scaleFactor
                                clip: true

                                Row {
                                    spacing: 10

                                    Image {
                                        width: symbolWidth
                                        height: symbolHeight
                                        source: symbolUrl
                                        anchors.verticalCenter: parent.verticalCenter
                                    }
                                    Text {
                                        width: 125 * scaleFactor
                                        text: name
                                        wrapMode: Text.WordWrap
                                        font.pixelSize: 12 * scaleFactor
                                        anchors.verticalCenter: parent.verticalCenter
                                    }
                                }
                            }

                            section {
                                property: "layerName"
                                criteria: ViewSection.FullString
                                labelPositioning: ViewSection.CurrentLabelAtStart | ViewSection.InlineLabels
                                delegate: Rectangle {
                                    width: 180 * scaleFactor
                                    height: childrenRect.height
                                    color: "lightsteelblue"

                                    Text {
                                        text: section
                                        font.bold: true
                                        font.pixelSize: 13 * scaleFactor
                                    }
                                }
                            }
// Necessary to close the code
                        }
                    }
                }
            }
        }
    }

    Controls.DescriptionPage{
        id:descPage
        visible: false
    }
}
```
### That's it! More examples can be found on the [ESRI AppStudio Github](https://github.com/Esri/arcgis-appstudio-samples "ESRI Samples")


