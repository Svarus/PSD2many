//var inFolder = new Folder("D:/GaddddddmingX/Script/");

var wasFound = false;
var inputFolderPath = "";
var layerList = new Array() ;

showDialog();

///________________The End_____________________________



//______________additional funcs__________________________


function getLayers(docElement) {
    var len = docElement.layerSets.length;
    var element;
    var lenn;
    var pushLayer;
    for(var a = 0; a < len; a++) {        
        element = docElement.layerSets[a];        
        lenn = element.layerSets.length;
        if( lenn > 0){
            // recursive
            //recursLayer = docElement.layerSets[a];
            layerList.push(element) ;  
            getLayers(element);                
        } else {
            //pushLayer = docElement.layerSets[a]
            layerList.push(element) ;           
            //return;
            }
        }
    };

function selectLayerSet(docElement, nameFind) {
    // find layer groups
    for(var a = 0; a < docElement.layerSets.length; a++) {
        var currentName = docElement.layerSets[a].name;
        if (currentName == nameFind) {
            //saveLayer(el.layers.getByName(lname), lname, oldPath, true);
            activeDocument.activeLayer = docElement.layerSets[a];
            placeBeforeLayerSet = docElement.layerSets[a];
            wasFound = true;
            return;
        } else {
            // recursive
            selectLayerSet(docElement.layerSets[a], nameFind);
            }
        }
    };
 
 function savePSD(saveFile){
    psdSaveOptions = new PhotoshopSaveOptions();
    psdSaveOptions.embedColorProfile = true;
    psdSaveOptions.alphaChannels = true;
    activeDocument.saveAs(saveFile, psdSaveOptions, true, Extension.LOWERCASE);
    };

function savePNG(saveFile, isTranspared){
    var pngOpts = new ExportOptionsSaveForWeb; 
    pngOpts.format = SaveDocumentType.PNG
    pngOpts.PNG8 = false;   //PNG = 24
    pngOpts.transparency = isTranspared; 
    pngOpts.interlaced = false; 
    pngOpts.quality = 100;
    activeDocument.exportDocument(new File(saveFile), ExportType.SAVEFORWEB, pngOpts); 
};

function makeActiveByIndex( idx, forceVisible,add ){
    //add == undefined ? add = false : add = true
     try{
          var desc = new ActionDescriptor();
          var ref = new ActionReference();
          ref.putIndex(charIDToTypeID( "Lyr " ), idx)
          desc.putReference( charIDToTypeID( "null" ), ref );
          if(add) desc.putEnumerated( stringIDToTypeID( "selectionModifier" ), stringIDToTypeID( "selectionModifierType" ), stringIDToTypeID( "addToSelection" ) );
          desc.putBoolean( charIDToTypeID( "MkVs" ), forceVisible );
          executeAction( charIDToTypeID( "slct" ), desc, DialogModes.NO );
     }catch(e){ return -1;}
};

function doWork(folderIcons, inFolder, prefix){
    //var inFolder = "C:/Users/skyrylenko/Desktop/PSD/psd/";
    var inFile = "";
    //var folderIcons = "Amaya";
                           
    var games = []; 
    var psdFiles = [];
    var psdFilesSize = [];
        
    var originalUnit = preferences.rulerUnits
    preferences.rulerUnits = Units.PIXELS
    displayDialogs = DialogModes.NO

    var resolution = 72;
    var resampleMethod = ResampleMethod.AUTOMATIC;
    var placeBeforeLayerSet;    //should be global

    //var fileInput = File(inFolder + inFile);
    var sourceDoc = activeDocument;
    var savedStateBeginning = sourceDoc.activeHistoryState;
    var savedState = sourceDoc.activeHistoryState;

    //create array of games 
    selectLayerSet(sourceDoc, folderIcons); 
    if (wasFound){
        var gamesSet = sourceDoc.activeLayer;
        var maxCount = sourceDoc.activeLayer.layerSets.length;
        for(var i = 0; i < maxCount; i++) {
             games[i] = sourceDoc.activeLayer.layerSets[i].name;
             }
         
        for (var count = 0; count < maxCount ; count++){
            //delete all games except first
            var layer;
            if (count < maxCount - 1){
                layer = sourceDoc.activeLayer.layerSets[1];
                makeActiveByIndex( layer.itemIndex, false, false );
                for (var i = 2; i < games.length; i++){
                    layer = gamesSet.layerSets[i];
                    makeActiveByIndex( layer.itemIndex, false, true );
                    }    
                activeDocument.activeLayer.remove();
                }   

            //set visibility of game to visible    
            selectLayerSet(activeDocument, games[0]); 
            activeDocument.activeLayer.visible = true;  
           
            //save PSD file    
            var nameStr = (inFolder + "/"  + prefix + games[0]).replace(/\.[^\.]+$/, '');   
            
            var saveFile = new File(nameStr);
            //saveFile = saveFile.replace(/\.[^\.]+$/, '');
            savePSD(saveFile);    

            //restore opened document
            activeDocument.activeHistoryState = savedState;

            //delete saved game and update history state
            selectLayerSet(activeDocument, games[0]); 
            activeDocument.activeLayer.remove();
            savedState = activeDocument.activeHistoryState;
            
            //update array of games 
            games = [];
            selectLayerSet(sourceDoc, folderIcons); 
            for(var i = 0; i < sourceDoc.activeLayer.layerSets.length; i++) {
                games[i] = sourceDoc.activeLayer.layerSets[i].name;
                }
            }

        //restore document and clear history
        try {
            activeDocument.activeHistoryState = savedStateBeginning;
            app.purge(PurgeTarget.HISTORYCACHES);
            }
        catch(e){};

        // Restore original ruler unit setting
        app.preferences.rulerUnits = originalUnit;
        }    
    else {
        alert("Can't find folder \"" + folderIcons + "\". Please choose correct folder name...");
        showDialog();
        }
};

function showDialog(){
    var win = new Window("dialog");
    g = win.graphics;    
    //var myBrush = g.newBrush(g.BrushType.SOLID_COLOR, [0, 0, 0, 1]);
    //g.backgroundColor = myBrush;
    win.title = win.add('statictext', undefined, 'Exctract set of folders to PSD files');
    var g = win.title.graphics;
    g.font = ScriptUI.newFont("Arial","BOLDITALIC", 22);    

    //win.tree1 = win.add('treeview', [0, 0, 150, 150]);
    //win.tree1.node = win.tree1.add("node", "node1");
    /*var tree = win.add("treeview", [0, 0, 50, 50]);
    var mammals = tree.add("node", "node1");
    mammals.add("item", "cats");    
    mammals.add("item", "dogs");  
    var ins = tree.add("node", "node1");
    ins.add("item", "ants");    
    ins.add("item", "poes");    
    mammals.expanded = true;*/
    
    win.p1  = win.add("panel", undefined, undefined, {borderStyle:'black'});
    win.p2  = win.add("panel", undefined, undefined, {borderStyle:'none'});
    //win.p1.grp0 =  win.p1.add("group");
    //win.p1.grp0.orientation='row';
    //win.p1.grp0.alignChildren='right';
    //win.p1.grp0.spacing=10;
    //win.p1.grp0.rb1 = win.p1.grp0.add('radiobutton',undefined,'Case Sensitive');
    //win.p1.grp0.rb2 = win.p1.grp0.add('radiobutton',undefined,'Not Case Sensitive');
    //win.p1.grp0.cb1 = win.p1.grp0.add('checkbox',undefined,'Select all occurences');
    //win.p1.grp0.rb1.value=true;
    //win.p1.grp0.cb1.value=true;
    

    /*win.p1.grp6 =  win.p1.add("group");
    win.p1.grp6.orientation = 'row';
    win.p1.grp6.alignment = 'right';
    win.p1.grp6.spacing = 10;
    win.p1.grp6.st1 = win.p1.grp6.add('statictext', undefined, 'Choose PSD file to open:');    
    win.p1.grp6.bu1 = win.p1.grp6.add('button', undefined, 'Browse...');
    win.p1.grp6.bu1.preferredSize = [100, 25];    
    win.p1.grp6.bu1.onClick = function(){
        var myFile = File.openDialog("Please select the input PSD file");
        if (myFile != null){
            win.p1.grp7.st1.text = myFile.absoluteURI.toString();
            win.p1.grp7.st1.helpTip = myFile.absoluteURI.toString();
            inputFolderPath = myFile.absoluteURI;
            }        
        }  */
    
    win.p1.grp7 =  win.p1.add("group");
    win.p1.grp7.orientation = 'row';
    win.p1.grp7.alignment = 'fill';
    win.p1.grp7.spacing = 10;
    win.p1.grp7.st1 = win.p1.grp7.add('statictext', undefined, 'Choose PSD file:');   
    win.p1.grp7.ddlist1 = win.p1.grp7.add('dropdownlist',  undefined, app.documents);    
    //win.p1.grp7.ddlist1.selection = 1;
    win.p1.grp7.ddlist1.onChange = function() {
        activeDocument = app.documents[win.p1.grp7.ddlist1.selection.index];
        getLayers(activeDocument);
        //win.p1.grp1.ddlist1 = win.p1.grp1.add('dropdownlist',  undefined, layerList);  
        win.p1.grp1.ddlist1.removeAll();
        for ( var i = 0; i < layerList.length; i++ ) {
			win.p1.grp1.ddlist1.add( "item", layerList[i].name );
}
        };
    
    win.p1.grp0 =  win.p1.add("group");
    win.p1.grp0.orientation = 'row';
    win.p1.grp0.alignment = 'fill';
    win.p1.grp0.spacing = 10;
    win.p1.grp0.st1 = win.p1.grp0.add('statictext', undefined, 'Choose Folder to save files there:');    
    win.p1.grp0.bu1 = win.p1.grp0.add('button', undefined, 'Browse...');
    win.p1.grp0.bu1.preferredSize = [100, 25];    
    win.p1.grp0.bu1.onClick = function(){
        var path = "~";
        var defFolder = Folder(path);
        if ( !defFolder.exists )
        {
            defFolder = Folder("~");
        }
    //setting normal color
    win.p1.grp0.bu1.graphics.foregroundColor = win.p1.grp0.bu1.graphics.newPen(win.graphics.PenType.SOLID_COLOR, [0.83, 0.83, 0.83], 1);
    var folderName = Folder.selectDialog( "Please select the output folder for saving PSD files", defFolder );
    if (folderName != null && folderName != "~"){
        win.p1.grp3.st1.text = folderName.absoluteURI.toString();
        win.p1.grp3.st1.helpTip = folderName.absoluteURI.toString();
        inputFolderPath = folderName.absoluteURI;
        }        
    }  

    win.p1.grp5 =  win.p1.add("group");
    win.p1.grp5.orientation = 'row';
    win.p1.grp5.alignment = 'fill';
    win.p1.grp5.spacing = 10;
    win.p1.grp5.st1 = win.p1.grp5.add('statictext', undefined, 'Set Prefix for PSD files names:');
    win.p1.grp5.st1.helpTip = 'This will add this prefix to each saved PSD file';
    win.p1.grp5.et1 = win.p1.grp5.add('edittext');
    win.p1.grp5.et1.preferredSize = [300, 20];
    //win.p1.grp5.et1.active = true;
    
    win.p1.grp1 =  win.p1.add("group");
    win.p1.grp1.orientation = 'row';
    win.p1.grp1.alignment = 'fill';
    win.p1.grp1.spacing = 10;
    win.p1.grp1.st1 = win.p1.grp1.add('statictext', undefined, 'PSD Folder name to process:');
    win.p1.grp1.st1.helpTip = 'All subfolders of this folder will be created as separate PSD files';
    /*win.p1.grp1.et1 = win.p1.grp1.add('edittext');
    win.p1.grp1.et1.preferredSize = [300, 20];
    win.p1.grp1.et1.active = true;  */
    //var array = app.activeDocument.layers;   
   
    //getLayers(activeDocument);
    win.p1.grp1.ddlist1 = win.p1.grp1.add('dropdownlist',  undefined, layerList);  

    win.p1.grp3 =  win.p1.add("group");
    win.p1.grp3.orientation = 'row';
    win.p1.grp3.alignment = 'fill';
    win.p1.grp3.spacing = 10;
    var nn;
    if (inputFolderPath == null){
        nn = "";
        }
    else {
        nn = inputFolderPath;
        }
    
    win.p1.grp3.st1 = win.p1.grp3.add('statictext', undefined, nn);
    win.p1.grp3.st1.preferredSize = [400, 20];    
   
    win.p2.grp2 =  win.p2.add("group");
    win.p2.grp2.orientation = 'row';
    win.p2.grp2.spacing = 10;
    win.p2.grp2.bu1 = win.p2.grp2.add('button', undefined, 'Start');
    win.p2.grp2.bu1.preferredSize = [190, 25];
    win.p2.grp2.bu2 = win.p2.grp2.add('button', undefined, 'Cancel');
    win.p2.grp2.bu2.preferredSize = [190, 25];
    win.p2.grp2.bu1.onClick = function(){
        win.p1.grp1.st1.graphics.foregroundColor = win.p1.grp1.st1.graphics.newPen(win.graphics.PenType.SOLID_COLOR, [0.83, 0.83, 0.83], 1); 
        win.p1.grp7.st1.graphics.foregroundColor = win.p1.grp7.st1.graphics.newPen(win.graphics.PenType.SOLID_COLOR, [0.83, 0.83, 0.83], 1); 
        win.p1.grp1.st1.graphics.foregroundColor = win.p1.grp1.st1.graphics.newPen(win.graphics.PenType.SOLID_COLOR, [0.83, 0.83, 0.83], 1); 
        
        if (win.p1.grp3.st1.text == ''){            
            win.p1.grp0.bu1.graphics.foregroundColor = win.p1.grp0.bu1.graphics.newPen(win.graphics.PenType.SOLID_COLOR, [1, 0, 0], 1);
            alert('Please browse folder for saving');
            }

        if (win.p1.grp7.ddlist1.selection  == null){
            win.p1.grp7.st1.graphics.foregroundColor = win.p1.grp7.st1.graphics.newPen(win.graphics.PenType.SOLID_COLOR, [1, 0, 0], 1);
            alert('Please select PSD file');
            }
        if (win.p1.grp1.ddlist1.selection == null){
            win.p1.grp1.st1.graphics.foregroundColor = win.p1.grp1.st1.graphics.newPen(win.graphics.PenType.SOLID_COLOR, [1, 0, 0], 1);
            alert('Please select PSD folder name');
            }
       
        /*if(win.p1.grp1.et1.text == ''){
            alert('Please specify folder name for processing');
            win.p1.grp1.st1.graphics.foregroundColor = win.p1.grp1.st1.graphics.newPen(win.graphics.PenType.SOLID_COLOR, [1, 0, 0], 1);
            //win.p1.grp0.bu1.fillBrush = win.p1.grp0.bu1.graphics.newBrush( win.p1.grp0.bu1.graphics.BrushType.SOLID_COLOR, [0, 0, 0, 1] );
            //win.p1.grp1.et1.active = true;
            }*/
        if (win.p1.grp3.st1.text == '' || win.p1.grp7.ddlist1.selection  == null || win.p1.grp1.ddlist1.selection == null){
            return;
            }
        
        var nn = win.p1.grp7.ddlist1.selection.index;
        activeDocument = app.documents[nn];
        
        win.close(1);                
        doWork(win.p1.grp1.ddlist1.selection.text, win.p1.grp3.st1.text, win.p1.grp5.et1.text);
       }
    win.p2.grp2.bu2.onClick=function(){
            win.close(2);
    }

    win.center();
    win.show();
};
