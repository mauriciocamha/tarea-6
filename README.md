# tarea-6
Para hacerlo funcionar es necesario:

Tener Node instalado

Instalar localmente express con npm install express --save

Instalar localmente mongoose con npm install mongoose --save

Hacerse una cuenta en mlab.com (con un plan sandbox que es gratis) para tener acceso a una base de datos mongodb remota o algo que sea similar.

Crear un archivo local llamado credentials.js que contenga la url de conexión.

module.exports = { MONGOURL: 'mongodb+srv://[base]:[contrasena]@chat-qypoo.gcp.mongodb.net/test?retryWrites=true&w=majority' };

donde reemplazan los dos campos con la base de datos y la contraseña (sin los corchetes).

var express = require('express'),
	app = express(),
	server = require('http').createServer(app),
	io = require('socket.io').listen(server),
	nicknames = [];
	
app.use(express.static('public'));
var mongoose = require('mongoose');
var credentials = require('./credentials.js');
var dbUrl=credentials.MONGOURL;
mongoose.connect(dbUrl , {useNewUrlParser: true, useUnifiedTopology: true }, (err) => {
  console.log('mongodb connected',err);
})
var Message = mongoose.model('Message',{
  msg : String,
  nick : String,
  dateAdded : {type: Date, default: Date.now}
})
	
server.listen(3000);

app.get('/', function(req, res){
	res.sendFile(__dirname + '/index.html');
	});
	
io.sockets.on('connection', function(socket){
	socket.on('new user', function(data, callback){
		if (nicknames.indexOf(data) != -1) {
			callback(false);
		} else {
			callback(true);
			socket.nickname = data;
			nicknames.push(socket.nickname);
			updateNicknames();
			io.sockets.emit('is_online', '🔵 <i>' + socket.nickname + ' se unió a la conversación...</i><br/>');
		}
	});
	
	// recarga la lista de nicks
	function updateNicknames(){
		io.sockets.emit('usernames', nicknames);
	}
	
	// envía el nombre de usuario junto con el mensaje
	socket.on('send message', function(data){
		var message = new Message({msg: data, nick: socket.nickname});
		 message.save((err) =>{
			if(!err)
				io.sockets.emit('new message', message);
		  })
		
		
	});
	
	// controla la desconexión de los usuarios
	socket.on('disconnect', function(data){
		if(!socket.nickname) return;
		nicknames.splice(nicknames.indexOf(socket.nickname), 1);
		updateNicknames();
		io.sockets.emit('is_offline', '🔴 <i>' + socket.nickname + ' dejó la conversación...</i><br/>');
	});
});

/* Requires: normalize.css -box-sizing.htc */
/* Global Reset & Standards ---------------------- */
* { -webkit-box-sizing: border-box; -moz-box-sizing: border-box; box-sizing: border-box; }

html { font-size: 62.5%; }

body { background-image: url('../images/subtlenet2.png'); font-family: "Helvetica Neue", "HelveticaNeue", Helvetica, Arial, "Lucida Grande", sans-serif; font-size: 14px; font-size: 1.4rem; line-height: 1; color: #222222; position: relative; -webkit-font-smoothing: antialiased; }

/* Links ---------------------- */
a { color: #2ba6cb; text-decoration: none; line-height: inherit; }

a:hover { color: #2795b6; }

a:focus { color: #2ba6cb; outline: none; }

p a, p a:visited { line-height: inherit; }

/* Misc ---------------------- */
.left { float: left; }

.right { float: right; }

.text-left { text-align: left; }

.text-right { text-align: right; }

.text-center { text-align: center; }

.hide { display: none; }

.highlight { background: #ffff99; }

#googlemap img, object, embed { max-width: none; }

#map_canvas embed { max-width: none; }

#map_canvas img { max-width: none; }

#map_canvas object { max-width: none; }

/* Base Type Styles Using Modular Scale ---------------------- */
body, div, dl, dt, dd, ul, ol, li, h1, h2, h3, h4, h5, h6, pre, form, fieldset, p, blockquote, th, td { margin: 0; padding: 0; font-size: 14px; }

p { font-size: 14px; line-height: 1.6; margin-bottom: 17px; }
p.lead { font-size: 17.5px; line-height: 1.6; margin-bottom: 17px; }
p img.left, p img { margin: 17px; margin-left: 0; }
p img.right { margin: 17px; margin-right: 0; }

aside p { font-size: 13px; line-height: 1.35; font-style: italic; }

h1, h2, h3, h4, h5, h6 { text-rendering: optimizeLegibility; line-height: 1.1; margin-bottom: 14px; margin-top: 14px; }
h1 small, h2 small, h3 small, h4 small, h5 small, h6 small { font-size: 60%; color: #888; line-height: 0; }

h1 { font-size: 44px; }

h2 { font-size: 37px; }

h3 { font-size: 27px; }

h4 { font-size: 23px; }

h5 { font-size: 17px; }

h6 { font-size: 14px; }

hr { border: solid #ddd; border-width: 5px 0 0; clear: both; margin: 22px 0 21px; height: 0; }

.subheader { line-height: 1.3; color: #777; font-weight: 300; margin-bottom: 17px; }

em, i { font-style: italic; line-height: inherit; }

strong, b { font-weight: bold; line-height: inherit; }

small { font-size: 60%; line-height: inherit; }

code { font-weight: bold; background: #ffff99; }

/* Lists ---------------------- */
ul, ol { font-size: 14px; line-height: 1.6; margin-bottom: 17px; list-style-position: inside; }

ul.square, ul.circle, ul.disc { margin-left: 17px; }

ul.square { list-style-type: square; }

ul.circle { list-style-type: circle; }

ul.disc { list-style-type: disc; }

ul.no-bullet { list-style: none; }

ul.large li { line-height: 21px; }

/* Blockquotes ---------------------- */
blockquote, blockquote p { line-height: 1.5; color: #777; }

blockquote { margin: 0 0 17px; padding: 9px 20px 0 19px; border-left: 1px solid #ddd; }

blockquote cite { display: block; font-size: 13px; color: #555; }

blockquote cite:before { content: "\2014 \0020"; }

blockquote cite a, blockquote cite a:visited { color: #555; }

abbr, acronym { text-transform: uppercase; font-size: 90%; color: #222; border-bottom: 1px solid #ddd; cursor: help; }

abbr { text-transform: none; }

/* Print styles.  Inlined to avoid required HTTP connection: www.phpied.com/delay-loading-your-print-css/ Credit to Paul Irish and HTML5 Boilerplate (html5boilerplate.com)
*/
.print-only { display: none !important; }

@media print { * { background: transparent !important; color: black !important; box-shadow: none !important; text-shadow: none !important; filter: none !important; -ms-filter: none !important; }
  /* Black prints faster: h5bp.com/s */
  a, a:visited { text-decoration: underline; }
  a[href]:after { content: " (" attr(href) ")"; }
  abbr[title]:after { content: " (" attr(title) ")"; }
  .ir a:after, a[href^="javascript:"]:after, a[href^="#"]:after { content: ""; }
  /* Don't show links for images, or javascript/internal links */
  pre, blockquote { border: 1px solid #999; page-break-inside: avoid; }
  thead { display: table-header-group; }
  /* h5bp.com/t */
  tr, img { page-break-inside: avoid; }
  img { max-width: 100% !important; }
  @page { margin: 0.5cm; }
  p, h2, h3 { orphans: 3; widows: 3; }
  h2, h3 { page-break-after: avoid; }
  .hide-on-print { display: none !important; }
  .print-only { display: block !important; } }

/* The Grid ---------------------- */
.row { width: 1000px; max-width: 100%; min-width: 768px; margin: 0 auto; }
.row .row { width: auto; max-width: none; min-width: 0; margin: 0 -15px; }
.row.collapse .column, .row.collapse .columns { padding: 0; }
.row .row { width: auto; max-width: none; min-width: 0; margin: 0 -15px; }
.row .row.collapse { margin: 0; }

.column, .columns { float: left; min-height: 1px; padding: 0 15px; position: relative; }
.column.centered, .columns.centered { float: none; margin: 0 auto; }

[class*="column"] + [class*="column"]:last-child { float: right; }

[class*="column"] + [class*="column"].end { float: left; }

.row .one { width: 8.333%; }

.row .two { width: 16.667%; }

.row .three { width: 25%; }

.row .four { width: 33.333%; }

.row .five { width: 41.667%; }

.row .six { width: 50%; }

.row .seven { width: 58.333%; }

.row .eight { width: 66.667%; }

.row .nine { width: 75%; }

.row .ten { width: 83.333%; }

.row .eleven { width: 91.667%; }

.row .twelve { width: 100%; }

.row .offset-by-one { margin-left: 8.333%; }

.row .offset-by-two { margin-left: 16.667%; }

.row .offset-by-three { margin-left: 25%; }

.row .offset-by-four { margin-left: 33.333%; }

.row .offset-by-five { margin-left: 41.667%; }

.row .offset-by-six { margin-left: 50%; }

.row .offset-by-seven { margin-left: 58.333%; }

.row .offset-by-eight { margin-left: 66.667%; }

.row .offset-by-nine { margin-left: 75%; }

.row .offset-by-ten { margin-left: 83.333%; }

.push-two { left: 16.667%; }

.pull-two { right: 16.667%; }

.push-three { left: 25%; }

.pull-three { right: 25%; }

.push-four { left: 33.333%; }

.pull-four { right: 33.333%; }

.push-five { left: 41.667%; }

.pull-five { right: 41.667%; }

.push-six { left: 50%; }

.pull-six { right: 50%; }

.push-seven { left: 58.333%; }

.pull-seven { right: 58.333%; }

.push-eight { left: 66.667%; }

.pull-eight { right: 66.667%; }

.push-nine { left: 75%; }

.pull-nine { right: 75%; }

.push-ten { left: 83.333%; }

.pull-ten { right: 83.333%; }

img, object, embed { max-width: 100%; height: auto; padding: 5px 5px 5px 5px;}

img { -ms-interpolation-mode: bicubic; }

#map_canvas img, .map_canvas img { max-width: none!important; }

/* Nicolas Gallagher's micro clearfix */
.row { *zoom: 1; }
.row:before, .row:after { content: ""; display: table; }
.row:after { clear: both; }

/* Mobile Grid and Overrides ---------------------- */
@media only screen and (max-width: 767px) { body { -webkit-text-size-adjust: none; -ms-text-size-adjust: none; width: 100%; min-width: 0; margin-left: 0; margin-right: 0; padding-left: 0; padding-right: 0; }
  .row { width: auto; min-width: 0; margin-left: 0; margin-right: 0; }
  .column, .columns { width: auto !important; float: none; }
  .column:last-child, .columns:last-child { float: none; }
  [class*="column"] + [class*="column"]:last-child { float: none; }
  .column:before, .columns:before, .column:after, .columns:after { content: ""; display: table; }
  .column:after, .columns:after { clear: both; }
  .no-left-margin, .offset-by-one, .offset-by-two, .offset-by-three, .offset-by-four, .offset-by-five, .offset-by-six, .offset-by-seven, .offset-by-eight, .offset-by-nine, .offset-by-ten { margin-left: 0 !important; }
  .left-auto, .push-two, .push-three, .push-four, .push-five, .push-six, .push-seven, .push-eight, .push-nine, .push-ten { left: auto; }
  .right-auto, .pull-two, .pull-three, .pull-four, .pull-five, .pull-six, .pull-seven, .pull-eight, .pull-nine, .pull-ten { right: auto; }
  /* Mobile 4-column Grid */
  .row .mobile-one { width: 25% !important; float: left; padding: 0 15px; }
  .row .mobile-one:last-child { float: right; }
  .row.collapse .mobile-one { padding: 0; }
  .row .mobile-two { width: 50% !important; float: left; padding: 0 15px; }
  .row .mobile-two:last-child { float: right; }
  .row.collapse .mobile-two { padding: 0; }
  .row .mobile-three { width: 75% !important; float: left; padding: 0 15px; }
  .row .mobile-three:last-child { float: right; }
  .row.collapse .mobile-three { padding: 0; }
  .row .mobile-four { width: 100% !important; float: left; padding: 0 15px; }
  .row .mobile-four:last-child { float: right; }
  .row.collapse .mobile-four { padding: 0; }
  .push-one-mobile { left: 25%; }
  .pull-one-mobile { right: 25%; }
  .push-two-mobile { left: 50%; }
  .pull-two-mobile { right: 50%; }
  .push-three-mobile { left: 75%; }
  .pull-three-mobile { right: 75%; } }
/* Block Grids ---------------------- */
/* These are 2-up, 3-up, 4-up and 5-up ULs, suited
for repeating blocks of content. Add 'mobile' to
them to switch them just like the layout grid
(one item per line) on phones
For IE7/8 compatibility block-grid items need to be
the same height. You can optionally uncomment the
lines below to support arbitrary height, but know
that IE7/8 do not support :nth-child.
-------------------------------------------------- */
.block-grid { display: block; overflow: hidden; padding: 0; }
.block-grid > li { display: block; height: auto; float: left; }

.block-grid.two-up { margin: 0 -15px; }

.block-grid.two-up > li { width: 50%; padding: 0 15px 15px; }

/*  .block-grid.two-up>li:nth-child(2n+1) {clear: left;} */
.block-grid.three-up { margin: 0 -12px; }

.block-grid.three-up > li { width: 33.33%; padding: 0 12px 12px; }

/*  .block-grid.three-up>li:nth-child(3n+1) {clear: left;} */
.block-grid.four-up { margin: 0 -10px; }

.block-grid.four-up > li { width: 25%; padding: 0 10px 10px; }

/*  .block-grid.four-up>li:nth-child(4n+1) {clear: left;} */
.block-grid.five-up { margin: 0 -8px; }

.block-grid.five-up > li { width: 20%; padding: 0 8px 8px; }

/*  .block-grid.five-up>li:nth-child(5n+1) {clear: left;} */
/* Mobile Block Grids */
@media only screen and (max-width: 767px) { .block-grid.mobile { margin-left: 0; }
  .block-grid.mobile > li { float: none; width: 100%; margin-left: 0; } }

/* Requires: globals.css */
/* Table of Contents
:: Visibility
:: Alerts
:: Labels
:: Tooltips
:: Panels
:: Side Nav
:: Sub Nav
:: Pagination
:: Breadcrumbs
:: Lists
:: Link Lists
:: Keystroke Chars
:: Video
:: Tables
:: Microformats
*/
/* Visibility Classes ---------------------- */
/* Standard visibility targeting */
.show-for-small, .show-for-medium, .hide-for-large, .show-for-xlarge { display: none !important; }

.hide-for-xlarge, .show-for-large, .hide-for-small, .hide-for-medium { display: block !important; }

/* Very large display targeting */
@media only screen and (min-width: 1441px) { .hide-for-small, .hide-for-medium, .hide-for-large, .show-for-xlarge { display: block !important; }
  .show-for-small, .show-for-medium, .show-for-large, .hide-for-xlarge { display: none !important; } }
/* Medium display targeting */
@media only screen and (max-width: 1279px) and (min-width: 768px) { .hide-for-small, .show-for-medium, .hide-for-large, .hide-for-xlarge { display: block !important; }
  .show-for-small, .hide-for-medium, .show-for-large, .show-for-xlarge { display: none !important; } }
/* Small display targeting */
@media only screen and (max-width: 767px) { .show-for-small, .hide-for-medium, .hide-for-large, .hide-for-xlarge { display: block !important; }
  .hide-for-small, .show-for-medium, .show-for-large, .show-for-xlarge { display: none !important; } }
/* Orientation targeting */
.show-for-landscape, .hide-for-portrait { display: block !important; }

.hide-for-landscape, .show-for-portrait { display: none !important; }

@media screen and (orientation: landscape) { .show-for-landscape, .hide-for-portrait { display: block !important; }
  .hide-for-landscape, .show-for-portrait { display: none !important; } }
@media screen and (orientation: portrait) { .show-for-portrait, .hide-for-landscape { display: block !important; }
  .hide-for-portrait, .show-for-landscape { display: none !important; } }
/* Touch-enabled device targeting */
.show-for-touch { display: none !important; }

.hide-for-touch { display: block !important; }

.touch .show-for-touch { display: block !important; }

.touch .hide-for-touch { display: none !important; }

/* Specific overrides for elements that require something other than display: block */
table.show-for-xlarge, table.show-for-large, table.hide-for-small, table.hide-for-medium { display: table !important; }

@media only screen and (max-width: 1279px) and (min-width: 768px) { .touch table.hide-for-xlarge, .touch table.hide-for-large, .touch table.hide-for-small, .touch table.show-for-medium { display: table !important; } }
@media only screen and (max-width: 767px) { table.hide-for-xlarge, table.hide-for-large, table.hide-for-medium, table.show-for-small { display: table !important; } }
/* Alerts ---------------------- */
div.alert-box { display: block; padding: 6px 7px 7px; font-weight: bold; font-size: 14px; color: white; background-color: #2ba6cb; border: 1px solid rgba(0, 0, 0, 0.1); margin-bottom: 12px; -webkit-border-radius: 3px; -moz-border-radius: 3px; -ms-border-radius: 3px; -o-border-radius: 3px; border-radius: 3px; text-shadow: 0 -1px rgba(0, 0, 0, 0.3); position: relative; }
div.alert-box.success { background-color: #5da423; color: #fff; text-shadow: 0 -1px rgba(0, 0, 0, 0.3); }
div.alert-box.alert { background-color: #c60f13; color: #fff; text-shadow: 0 -1px rgba(0, 0, 0, 0.3); }
div.alert-box.secondary { background-color: #e9e9e9; color: #505050; text-shadow: 0 1px rgba(255, 255, 255, 0.3); }
div.alert-box a.close { color: #333; position: absolute; right: 4px; top: -1px; font-size: 17px; opacity: 0.2; padding: 4px; }
div.alert-box a.close:hover, div.alert-box a.close:focus { opacity: 0.4; }

/* Labels ---------------------- */
.label { padding: 1px 4px 2px; font-size: 12px; font-weight: bold; text-align: center; text-decoration: none; line-height: 1; white-space: nowrap; display: inline; position: relative; bottom: 1px; color: #fff; background: #2ba6cb; }
.label.radius { -webkit-border-radius: 3px; -moz-border-radius: 3px; -ms-border-radius: 3px; -o-border-radius: 3px; border-radius: 3px; }
.label.round { padding: 1px 7px 2px; -webkit-border-radius: 1000px; -moz-border-radius: 1000px; -ms-border-radius: 1000px; -o-border-radius: 1000px; border-radius: 1000px; }
.label.alert { background-color: #c60f13; }
.label.success { background-color: #5da423; }
.label.secondary { background-color: #e9e9e9; color: #505050; }

/* Tooltips ---------------------- */
.has-tip { border-bottom: dotted 1px #ccc; cursor: help; font-weight: bold; color: #333; }
.has-tip:hover { border-bottom: dotted 1px #0593dc; color: #0192dd; }
.has-tip.tip-left, .has-tip.tip-right { float: none !important; }

.tooltip { display: none; background: black; background: rgba(0, 0, 0, 0.8); position: absolute; color: #fff; font-weight: bold; font-size: 12px; font-size: 1.2rem; padding: 5px; z-index: 999; -webkit-border-radius: 4px; -moz-border-radius: 4px; border-radius: 4px; line-height: normal; }
.tooltip > .nub { display: block; width: 0; height: 0; border: solid 5px; border-color: transparent transparent black transparent; border-color: transparent transparent rgba(0, 0, 0, 0.8) transparent; position: absolute; top: -10px; left: 10px; }
.tooltip.tip-override > .nub { border-color: transparent transparent black transparent !important; border-color: transparent transparent rgba(0, 0, 0, 0.8) transparent !important; top: -10px !important; }
.tooltip.tip-top > .nub { border-color: black transparent transparent transparent; border-color: rgba(0, 0, 0, 0.8) transparent transparent transparent; top: auto; bottom: -10px; }
.tooltip.tip-left, .tooltip.tip-right { float: none !important; }
.tooltip.tip-left > .nub { border-color: transparent transparent transparent black; border-color: transparent transparent transparent rgba(0, 0, 0, 0.8); right: -10px; left: auto; }
.tooltip.tip-right > .nub { border-color: transparent black transparent transparent; border-color: transparent rgba(0, 0, 0, 0.8) transparent transparent; right: auto; left: -10px; }
.tooltip.noradius { -webkit-border-radius: 0; -moz-border-radius: 0; -ms-border-radius: 0; -o-border-radius: 0; border-radius: 0; }
.tooltip.opened { color: #0192DD !important; border-bottom: dotted 1px #0593DC !important; }

.tap-to-close { display: block; font-size: 10px; font-size: 1rem; color: #888; font-weight: normal; }

@media only screen and (max-width: 767px) { .tooltip { font-size: 14px; font-size: 1.4rem; line-height: 1.4; padding: 7px 10px 9px 10px; }
  .tooltip > .nub, .tooltip.top > .nub, .tooltip.left > .nub, .tooltip.right > .nub { border-color: transparent transparent black transparent; border-color: transparent transparent rgba(0, 0, 0, 0.85) transparent; top: -12px; left: 10px; } }
/* Panels ---------------------- */
div.panel { background: #f2f2f2; border: solid 1px #e6e6e6; margin: 0 0 22px 0; padding: 20px; }
div.panel *:first-child { margin-top: 0; }
div.panel *:last-child { margin-bottom: 0; }
div.panel.callout { background: #2ba6cb; color: #fff; border-color: #2284a1; -webkit-box-shadow: inset 0px 1px 0px rgba(255, 255, 255, 0.5); -moz-box-shadow: inset 0px 1px 0px rgba(255, 255, 255, 0.5); box-shadow: inset 0px 1px 0px rgba(255, 255, 255, 0.5); }
div.panel.callout a { color: #fff; }
div.panel.callout .button { background: white; border: none; color: #2ba6cb; text-shadow: none; }
div.panel.callout .button:hover { background: rgba(255, 255, 255, 0.8); }
div.panel.radius { -webkit-border-radius: 3px; -moz-border-radius: 3px; -ms-border-radius: 3px; -o-border-radius: 3px; border-radius: 3px; }

/* Side Nav ---------------------- */
ul.side-nav { display: block; list-style: none; margin: 0; padding: 17px 0; }
ul.side-nav li { display: block; list-style: none; margin: 0 0 7px 0; }
ul.side-nav li a { display: block; }
ul.side-nav li.active a { color: #4d4d4d; font-weight: bold; }
ul.side-nav li.divider { border-top: 1px solid #e6e6e6; height: 0; padding: 0; }

/* Sub Navs http://www.zurb.com/article/292/how-to-create-simple-and-effective-sub-na ---------------------- */
dl.sub-nav { display: block; width: auto; overflow: hidden; margin: -4px 0 18px -9px; padding-top: 4px; }
dl.sub-nav dt, dl.sub-nav dd { float: left; display: inline; margin-left: 9px; margin-bottom: 4px; }
dl.sub-nav dt { color: #999; font-weight: normal; }
dl.sub-nav dd a { text-decoration: none; -webkit-border-radius: 1000px; -moz-border-radius: 1000px; -ms-border-radius: 1000px; -o-border-radius: 1000px; border-radius: 1000px; }
dl.sub-nav dd.active a { font-weight: bold; background: #2ba6cb; color: #fff; padding: 3px 9px; cursor: default; }

/* Pagination ---------------------- */
ul.pagination { display: block; height: 24px; margin-left: -5px; }
ul.pagination li { float: left; display: block; height: 24px; color: #999; font-size: 14px; margin-left: 5px; }
ul.pagination li a { display: block; padding: 1px 7px 1px; color: #555; }
ul.pagination li:hover a, ul.pagination li a:focus { background: #e6e6e6; }
ul.pagination li.unavailable a { cursor: default; color: #999; }
ul.pagination li.unavailable:hover a, ul.pagination li.unavailable a:focus { background: transparent; }
ul.pagination li.current a { background: #2ba6cb; color: white; font-weight: bold; cursor: default; }
ul.pagination li.current a:hover { background: #2ba6cb; }

/* Breadcrums ---------------------- */
ul.breadcrumbs { display: block; background: #f6f6f6; padding: 6px 10px 7px; border: 1px solid #e9e9e9; -webkit-border-radius: 2px; -moz-border-radius: 2px; -ms-border-radius: 2px; -o-border-radius: 2px; border-radius: 2px; overflow: hidden; }
ul.breadcrumbs li { margin: 0; padding: 0 12px 0 0; float: left; list-style: none; }
ul.breadcrumbs li a, ul.breadcrumbs li span { text-transform: uppercase; font-size: 11px; font-size: 1.1rem; padding-left: 12px; }
ul.breadcrumbs li:first-child a, ul.breadcrumbs li:first-child span { padding-left: 0; }
ul.breadcrumbs li:before { content: "/"; color: #aaa; }
ul.breadcrumbs li:first-child:before { content: " "; }
ul.breadcrumbs li.current a { cursor: default; color: #333; }
ul.breadcrumbs li:hover a, ul.breadcrumbs li a:focus { text-decoration: underline; }
ul.breadcrumbs li.current:hover a, ul.breadcrumbs li.current a:focus { text-decoration: none; }
ul.breadcrumbs li.unavailable a { color: #999; }
ul.breadcrumbs li.unavailable:hover a, ul.breadcrumbs li.unavailable a:focus { text-decoration: none; color: #999; cursor: default; }

/* Lists ---------------------- */
ul.nice, ol.nice { list-style: none; margin: 0; }
ul.nice li, ol.nice li { padding-left: 13px; position: relative; }
ul.nice li span.bullet, ul.nice li span.number, ol.nice li span.bullet, ol.nice li span.number { position: absolute; left: 0; top: 0; color: #ccc; }

/* Link List */
ul.link-list { margin: 0 0 17px -22px; padding: 0; list-style: none; overflow: hidden; }
ul.link-list li { list-style: none; float: left; margin-left: 22px; display: block; }
ul.link-list li a { display: block; }

/* Keytroke Characters ---------------------- */
.keystroke, kbd { font-family: "Consolas", "Menlo", "Courier", monospace; font-size: 13px; padding: 2px 4px 0px; margin: 0; background: #ededed; border: solid 1px #dbdbdb; -webkit-border-radius: 3px; -moz-border-radius: 3px; -ms-border-radius: 3px; -o-border-radius: 3px; border-radius: 3px; }

/* Video - Mad props to http://www.alistapart.com/articles/creating-intrinsic-ratios-for-video/ ---------------------- */
.flex-video { position: relative; padding-top: 25px; padding-bottom: 67.5%; height: 0; margin-bottom: 16px; overflow: hidden; }
.flex-video.widescreen { padding-bottom: 57.25%; }
.flex-video.vimeo { padding-top: 0; }
.flex-video iframe, .flex-video object, .flex-video embed, .flex-video video { position: absolute; top: 0; left: 0; width: 100%; height: 100%; }

@media only screen and (max-device-width: 800px), only screen and (device-width: 1024px) and (device-height: 600px), only screen and (width: 1280px) and (orientation: landscape), only screen and (device-width: 800px), only screen and (max-width: 767px) { .flex-video { padding-top: 0; } }
/* Tables ---------------------- */
table { background: #fff; -moz-border-radius: 3px; -webkit-border-radius: 3px; border-radius: 3px; margin: 0 0 18px; border: 1px solid #ddd; }

table thead, table tfoot { background: #f5f5f5; }

table thead tr th, table tfoot tr th, table tbody tr td, table tr td, table tfoot tr td { font-size: 12px; font-size: 1.2rem; line-height: 18px; text-align: left; }

table thead tr th, table tfoot tr td { padding: 8px 10px 9px; font-size: 14px; font-size: 1.4rem; font-weight: bold; color: #222; }

table thead tr th:first-child, table tfoot tr td:first-child { border-left: none; }

table thead tr th:last-child, table tfoot tr td:last-child { border-right: none; }

table tbody tr.even, table tbody tr.alt { background: #f9f9f9; }

table tbody tr:nth-child(even) { background: #f9f9f9; }

table tbody tr td { color: #333; padding: 9px 10px; vertical-align: top; border: none; }

/* Microformats ---------------------- */
ul.vcard { display: inline-block; margin: 0 0 12px 0; border: 1px solid #ddd; padding: 10px; }
ul.vcard li { margin: 0; display: block; }
ul.vcard li.fn { font-weight: bold; font-size: 15px; font-size: 1.5rem; }

p.vevent span.summary { font-weight: bold; }
p.vevent abbr { cursor: default; text-decoration: none; font-weight: bold; border: none; padding: 0 1px; }

/* Requires globals.css */
/* Normal Buttons ---------------------- */
.button { width: auto; background: #2ba6cb; border: 1px solid #1e728c; -webkit-box-shadow: 0 1px 0 rgba(255, 255, 255, 0.5) inset; -moz-box-shadow: 0 1px 0 rgba(255, 255, 255, 0.5) inset; box-shadow: 0 1px 0 rgba(255, 255, 255, 0.5) inset; color: white; cursor: pointer; display: inline-block; font-family: "Helvetica Neue", "HelveticaNeue", Helvetica, Arial, "Lucida Grande", sans-serif; font-size: 14px; font-weight: bold; line-height: 1; margin: 0; outline: none; padding: 10px 20px 11px; position: relative; text-align: center; text-decoration: none; -webkit-transition: background-color 0.15s ease-in-out; -moz-transition: background-color 0.15s ease-in-out; -o-transition: background-color 0.15s ease-in-out; transition: background-color 0.15s ease-in-out; /* Hovers */ /* Sizes */ /* Colors */ /* Radii */ /* Layout */ /* Disabled ---------- */ }
.button:hover { color: white; background-color: #2284a1; }
.button:active { -webkit-box-shadow: 0 1px 0 rgba(0, 0, 0, 0.2) inset; -moz-box-shadow: 0 1px 0 rgba(0, 0, 0, 0.2) inset; box-shadow: 0 1px 0 rgba(0, 0, 0, 0.2) inset; }
.button:focus { -webkit-box-shadow: 0 0 4px #2ba6cb, 0 1px 0 rgba(255, 255, 255, 0.5) inset; -moz-box-shadow: 0 0 4px #2ba6cb, 0 1px 0 rgba(255, 255, 255, 0.5) inset; box-shadow: 0 0 4px #2ba6cb, 0 1px 0 rgba(255, 255, 255, 0.5) inset; color: white; }
.button.large { font-size: 17px; padding: 15px 30px 16px; }
.button.medium { font-size: 14px; }
.button.small { font-size: 11px; padding: 7px 14px 8px; }
.button.tiny { font-size: 10px; padding: 5px 10px 6px; }
.button.expand { width: 100%; text-align: center; }
.button.primary { background-color: #2ba6cb; border: 1px solid #1e728c; }
.button.primary:hover { background-color: #2284a1; }
.button.primary:focus { -webkit-box-shadow: 0 0 4px #2ba6cb, 0 1px 0 rgba(255, 255, 255, 0.5) inset; -moz-box-shadow: 0 0 4px #2ba6cb, 0 1px 0 rgba(255, 255, 255, 0.5) inset; box-shadow: 0 0 4px #2ba6cb, 0 1px 0 rgba(255, 255, 255, 0.5) inset; }
.button.success { background-color: #5da423; border: 1px solid #396516; }
.button.success:hover { background-color: #457a1a; }
.button.success:focus { -webkit-box-shadow: 0 0 5px #5da423, 0 1px 0 rgba(255, 255, 255, 0.5) inset; -moz-box-shadow: 0 0 5px #5da423, 0 1px 0 rgba(255, 255, 255, 0.5) inset; box-shadow: 0 0 5px #5da423, 0 1px 0 rgba(255, 255, 255, 0.5) inset; }
.button.alert { background-color: #c60f13; border: 1px solid #7f0a0c; }
.button.alert:hover { background-color: #970b0e; }
.button.alert:focus { -webkit-box-shadow: 0 0 4px #c60f13, 0 1px 0 rgba(255, 255, 255, 0.5) inset; -moz-box-shadow: 0 0 4px #c60f13, 0 1px 0 rgba(255, 255, 255, 0.5) inset; box-shadow: 0 0 4px #c60f13, 0 1px 0 rgba(255, 255, 255, 0.5) inset; }
.button.secondary { background-color: #e9e9e9; color: #1d1d1d; border: 1px solid #c3c3c3; }
.button.secondary:hover { background-color: #d0d0d0; }
.button.secondary:focus { -webkit-box-shadow: 0 0 5px #e9e9e9, 0 1px 0 rgba(255, 255, 255, 0.5) inset; -moz-box-shadow: 0 0 5px #e9e9e9, 0 1px 0 rgba(255, 255, 255, 0.5) inset; box-shadow: 0 0 5px #e9e9e9, 0 1px 0 rgba(255, 255, 255, 0.5) inset; }
.button.radius { -webkit-border-radius: 3px; -moz-border-radius: 3px; -ms-border-radius: 3px; -o-border-radius: 3px; border-radius: 3px; }
.button.round { -webkit-border-radius: 1000px; -moz-border-radius: 1000px; -ms-border-radius: 1000px; -o-border-radius: 1000px; border-radius: 1000px; }
.button.full-width { width: 100%; text-align: center; padding-left: 0 !important; padding-right: !important; }
.button.left-align { text-align: left; text-indent: 12px; }
.button.disabled, .button[disabled] { opacity: 0.6; cursor: default; background: #2ba6cb; -webkit-box-shadow: none; -moz-box-shadow: none; box-shadow: none; }

/* Don't use native buttons on iOS */
input[type=submit].button, button.button { -webkit-appearance: none; }

@media only screen and (max-width: 767px) { .button { display: block; }
  button.button, input[type="submit"].button { width: 100%; padding-left: 0; padding-right: 0; } }
/* Correct FF button padding */
@-moz-document url-prefix() { button::-moz-focus-inner, input[type="reset"]::-moz-focus-inner, input[type="button"]::-moz-focus-inner, input[type="submit"]::-moz-focus-inner, input[type="file"] > input[type="button"]::-moz-focus-inner { border: none; padding: 0; }
  input[type="submit"].tiny.button { padding: 3px 10px 4px; }
  input[type="submit"].small.button { padding: 5px 14px 6px; }
  input[type="submit"].button, input[type=submit].medium.button { padding: 8px 20px 9px; }
  input[type="submit"].large.button { padding: 13px 30px 14px; } }

/* Buttons with Dropdowns ---------------------- */
.button.dropdown { position: relative; padding-right: 44px; /* Sizes */ /* Triangles */ /* Flyout List */ /* Split Dropdown Buttons */ }
.button.dropdown.large { padding-right: 60px; }
.button.dropdown.small { padding-right: 28px; }
.button.dropdown.tiny { padding-right: 20px; }
.button.dropdown:after { content: ""; display: block; width: 0; height: 0; border: solid 6px; border-color: white transparent transparent transparent; position: absolute; top: 50%; right: 20px; margin-top: -2px; }
.button.dropdown.large:after { content: ""; display: block; width: 0; height: 0; border: solid 7px; border-color: white transparent transparent transparent; margin-top: -3px; right: 30px; }
.button.dropdown.small:after { content: ""; display: block; width: 0; height: 0; border: solid 5px; border-color: white transparent transparent transparent; margin-top: -2px; right: 14px; }
.button.dropdown.tiny:after { content: ""; display: block; width: 0; height: 0; border: solid 4px; border-color: white transparent transparent transparent; margin-top: -1px; right: 10px; }
.button.dropdown > ul { -webkit-box-sizing: content-box; -moz-box-sizing: content-box; box-sizing: content-box; display: none; position: absolute; left: -1px; background: #fff; background: rgba(255, 255, 255, 0.95); list-style: none; margin: 0; padding: 0; border: 1px solid #cccccc; border-top: none; min-width: 100%; z-index: 40; }
.button.dropdown > ul li { cursor: pointer; padding: 0; min-height: 18px; line-height: 18px; margin: 0; white-space: nowrap; list-style: none; }
.button.dropdown > ul li a { display: block; color: #555; font-size: 13px; font-weight: normal; padding: 6px 14px; text-align: left; }
.button.dropdown > ul li:hover { background-color: #e3f4f9; color: #222; }
.button.dropdown > ul li.divider { min-height: 0; padding: 0; height: 1px; margin: 4px 0; background: #ededed; }
.button.dropdown.up > ul { border-top: 1px solid #cccccc; border-bottom: none; }
.button.dropdown ul.no-hover.show-dropdown { display: block !important; }
.button.dropdown:hover > ul.no-hover { display: none; }
.button.dropdown.split { padding: 0; position: relative; /* Sizes */ /* Triangle Spans */ /* Colors */ }
.button.dropdown.split:after { display: none; }
.button.dropdown.split:hover { background-color: #2ba6cb; }
.button.dropdown.split.alert:hover { background-color: #c60f13; }
.button.dropdown.split.success:hover { background-color: #5da423; }
.button.dropdown.split.secondary:hover { background-color: #e9e9e9; }
.button.dropdown.split > a { color: white; display: block; padding: 10px 50px 11px 20px; -webkit-transition: background-color 0.15s ease-in-out; -moz-transition: background-color 0.15s ease-in-out; -o-transition: background-color 0.15s ease-in-out; transition: background-color 0.15s ease-in-out; }
.button.dropdown.split > a:hover { background-color: #2284a1; }
.button.dropdown.split.large > a { padding: 15px 75px 16px 30px; }
.button.dropdown.split.small > a { padding: 7px 35px 8px 14px; }
.button.dropdown.split.tiny > a { padding: 5px 25px 6px 10px; }
.button.dropdown.split > span { background-color: #2ba6cb; position: absolute; right: 0; top: 0; height: 100%; width: 30px; border-left: 1px solid #1e728c; -webkit-box-shadow: 1px 1px 0 rgba(255, 255, 255, 0.5) inset; -moz-box-shadow: 1px 1px 0 rgba(255, 255, 255, 0.5) inset; box-shadow: 1px 1px 0 rgba(255, 255, 255, 0.5) inset; -webkit-transition: background-color 0.15s ease-in-out; -moz-transition: background-color 0.15s ease-in-out; -o-transition: background-color 0.15s ease-in-out; transition: background-color 0.15s ease-in-out; }
.button.dropdown.split > span:hover { background-color: #2284a1; }
.button.dropdown.split > span:after { content: ""; display: block; width: 0; height: 0; border: solid 6px; border-color: white transparent transparent transparent; position: absolute; top: 50%; left: 50%; margin-left: -6px; margin-top: -2px; }
.button.dropdown.split.large span { width: 45px; }
.button.dropdown.split.small span { width: 21px; }
.button.dropdown.split.tiny span { width: 15px; }
.button.dropdown.split.large span:after { content: ""; display: block; width: 0; height: 0; border: solid 7px; border-color: white transparent transparent transparent; margin-top: -3px; margin-left: -7px; }
.button.dropdown.split.small span:after { content: ""; display: block; width: 0; height: 0; border: solid 4px; border-color: white transparent transparent transparent; margin-top: -1px; margin-left: -4px; }
.button.dropdown.split.tiny span:after { content: ""; display: block; width: 0; height: 0; border: solid 3px; border-color: white transparent transparent transparent; margin-top: -1px; margin-left: -3px; }
.button.dropdown.split.alert > span { background-color: #c60f13; border-left-color: #7f0a0c; }
.button.dropdown.split.success > span { background-color: #5da423; border-left-color: #396516; }
.button.dropdown.split.secondary > span { background-color: #e9e9e9; border-left-color: #c3c3c3; }
.button.dropdown.split.alert > a:hover, .button.dropdown.split.alert > span:hover { background-color: #970b0e; }
.button.dropdown.split.success > a:hover, .button.dropdown.split.success > span:hover { background-color: #457a1a; }
.button.dropdown.split.secondary > a:hover, .button.dropdown.split.secondary > span:hover { background-color: #d0d0d0; }

/* Button Groups ---------------------- */
ul.button-group { list-style: none; padding: 0; margin: 0 0 12px; overflow: hidden; }
ul.button-group li { padding: 0; margin: 0 0 0 -1px; float: left; }
ul.button-group li:first-child { margin-left: 0; }
ul.button-group.radius li:first-child a.button, ul.button-group.radius li:first-child a.button.radius, ul.button-group.radius li:first-child a.button.rounded { -webkit-border-radius: 0px; -moz-border-radius: 0px; -ms-border-radius: 0px; -o-border-radius: 0px; border-radius: 0px; border-top-left-radius: 3px; border-bottom-left-radius: 3px; }
ul.button-group.radius li + li a.button, ul.button-group.radius li + li a.button.radius, ul.button-group.radius li + li a.button.rounded { border-radius: 0; }
ul.button-group.radius li:last-child a.button, ul.button-group.radius li:last-child a.button.radius, ul.button-group.radius li:last-child a.button.rounded { -webkit-border-radius: 0px; -moz-border-radius: 0px; -ms-border-radius: 0px; -o-border-radius: 0px; border-radius: 0px; border-top-right-radius: 3px; border-bottom-right-radius: 3px; }
ul.button-group.rounded li:first-child a.button, ul.button-group.rounded li:first-child a.button.radius, ul.button-group.rounded li:first-child a.button.rounded { -webkit-border-radius: 0px; -moz-border-radius: 0px; -ms-border-radius: 0px; -o-border-radius: 0px; border-radius: 0px; border-top-left-radius: 1000px; border-bottom-left-radius: 1000px; }
ul.button-group.rounded li + li a.button, ul.button-group.rounded li + li a.button.radius, ul.button-group.rounded li + li a.button.rounded { border-radius: 0; }
ul.button-group.rounded li:last-child a.button, ul.button-group.rounded li:last-child a.button.radius, ul.button-group.rounded li:last-child a.button.rounded { -webkit-border-radius: 0px; -moz-border-radius: 0px; -ms-border-radius: 0px; -o-border-radius: 0px; border-radius: 0px; border-top-right-radius: 1000px; border-bottom-right-radius: 1000px; }
ul.button-group.even a.button { width: 100%; }
ul.button-group.even.two-up li { width: 50%; }
ul.button-group.even.three-up li { width: 33.3%; }
ul.button-group.even.three-up li:first-child { width: 33.4%; }
ul.button-group.even.four-up li { width: 25%; }
ul.button-group.even.five-up li { width: 20%; }

div.button-bar { overflow: hidden; }
div.button-bar ul.button-group { float: left; margin-right: 8px; }
div.button-bar ul.button-group:last-child { margin-left: 0; }

/* Requires globals.css app.js */
/* Tabs ---------------------- */
dl.tabs { border-bottom: solid 1px #e6e6e6; display: block; height: 40px; padding: 0; margin-bottom: 20px; }
dl.tabs.contained { margin-bottom: 0; }
dl.tabs dt { color: #b3b3b3; cursor: default; display: block; float: left; font-size: 12px; height: 40px; line-height: 40px; padding: 0 9px 0 20px; width: auto; text-transform: uppercase; }
dl.tabs dt:first-child { padding: 0 9px 0 0; }
dl.tabs dd { display: block; float: left; padding: 0; margin: 0; }
dl.tabs dd a { color: #6f6f6f; display: block; font-size: 14px; height: 40px; line-height: 40px; padding: 0px 23.8px; }
dl.tabs dd.active { border-top: 3px solid #2ba6cb; margin-top: -3px; }
dl.tabs dd.active a { cursor: default; color: #3c3c3c; background: #fff; border-left: 1px solid #e6e6e6; border-right: 1px solid #e6e6e6; font-weight: bold; }
dl.tabs dd:first-child { margin-left: 0; }
dl.tabs.vertical { height: auto; border-bottom: 1px solid #e6e6e6; }
dl.tabs.vertical dt, dl.tabs.vertical dd { float: none; height: auto; }
dl.tabs.vertical dd { border-left: 3px solid #cccccc; }
dl.tabs.vertical dd a { background: #f2f2f2; border: none; border: 1px solid #e6e6e6; border-width: 1px 1px 0 0; color: #555; display: block; font-size: 14px; height: auto; line-height: 1; padding: 15px 20px; -webkit-box-shadow: 0 1px 0 rgba(255, 255, 255, 0.5) inset; -moz-box-shadow: 0 1px 0 rgba(255, 255, 255, 0.5) inset; box-shadow: 0 1px 0 rgba(255, 255, 255, 0.5) inset; }
dl.tabs.vertical dd.active { margin-top: 0; border-top: 1px solid #4d4d4d; border-left: 4px solid #1a1a1a; }
dl.tabs.vertical dd.active a { background: #4d4d4d; border: none; color: #fff; height: auto; margin: 0; position: static; top: 0; -webkit-box-shadow: 0 0 0; -moz-box-shadow: 0 0 0; box-shadow: 0 0 0; }
dl.tabs.vertical dd:first-child a.active { margin: 0; }
dl.tabs.pill { border-bottom: none; margin-bottom: 10px; }
dl.tabs.pill dd { margin-right: 10px; }
dl.tabs.pill dd:last-child { margin-right: 0; }
dl.tabs.pill dd a { -webkit-border-radius: 1000px; -moz-border-radius: 1000px; -ms-border-radius: 1000px; -o-border-radius: 1000px; border-radius: 1000px; background: #e6e6e6; height: 26px; line-height: 26px; color: #666; }
dl.tabs.pill dd.active { border: none; margin-top: 0; }
dl.tabs.pill dd.active a { background-color: #2ba6cb; border: none; color: #fff; }
dl.tabs.pill.contained { border-bottom: solid 1px #eee; margin-bottom: 0; }
dl.tabs.two-up dt a, dl.tabs.two-up dd a, dl.tabs.three-up dt a, dl.tabs.three-up dd a, dl.tabs.four-up dt a, dl.tabs.four-up dd a, dl.tabs.five-up dt a, dl.tabs.five-up dd a { padding: 0 17px; text-align: center; overflow: hidden; }
dl.tabs.two-up dt, dl.tabs.two-up dd { width: 50%; }
dl.tabs.three-up dt, dl.tabs.three-up dd { width: 33.33%; }
dl.tabs.four-up dt, dl.tabs.four-up dd { width: 25%; }
dl.tabs.five-up dt, dl.tabs.five-up dd { width: 20%; }

ul.tabs-content { display: block; margin: 0 0 20px; padding: 0; }
ul.tabs-content > li { display: none; }
ul.tabs-content > li.active { display: block; }
ul.tabs-content.contained { padding: 0; }
ul.tabs-content.contained > li { border: solid 0 #e6e6e6; border-width: 0 1px 1px 1px; padding: 20px; }
ul.tabs-content.contained.vertical > li { border-width: 1px 1px 1px 1px; }

.no-js ul.tabs-content > li { display: block; }

@media only screen and (max-width: 767px) { dl.tabs.mobile, dl.nice.tabs.mobile { width: auto; margin: 20px -20px 40px; height: auto; }
  dl.tabs.mobile dt, dl.tabs.mobile dd, dl.nice.tabs.mobile dt, dl.nice.tabs.mobile dd { float: none; height: auto; }
  dl.tabs.mobile dd a { display: block; width: auto; height: auto; padding: 18px 20px; line-height: 1; border: solid 0 #ccc; border-width: 1px 0 0; margin: 0; color: #555; background: #eee; font-size: 15px; font-size: 1.5rem; }
  dl.tabs.mobile dd a.active { height: auto; margin: 0; border-width: 1px 0 0; }
  .tabs.mobile { border-bottom: solid 1px #ccc; height: auto; }
  .tabs.mobile dd a { padding: 18px 20px; border: none; border-left: none; border-right: none; border-top: 1px solid #ccc; background: #fff; }
  .tabs.mobile dd a.active { border: none; background: #2ba6cb; color: #fff; margin: 0; position: static; top: 0; height: auto; }
  .tabs.mobile dd:first-child a.active { margin: 0; }
  dl.contained.mobile, dl.nice.contained.mobile { margin-bottom: 0; }
  dl.contained.tabs.mobile dd a { padding: 18px 20px; }
  dl.tabs.mobile + ul.contained { margin-left: -20px; margin-right: -20px; border-width: 0 0 1px 0; } }

/* Requires globals.css */
.nav-bar { height: 40px; background: #4d4d4d; margin-top: 20px; padding: 0; }
.nav-bar > li { float: left; display: block; position: relative; padding: 0; margin: 0; border: 1px solid #333333; border-right: none; line-height: 38px; -webkit-box-shadow: 1px 0 0 rgba(255, 255, 255, 0.2) inset; -moz-box-shadow: 1px 0 0 rgba(255, 255, 255, 0.2) inset; box-shadow: 1px 0 0 rgba(255, 255, 255, 0.2) inset; }
.nav-bar > li:first-child { -webkit-box-shadow: 0 0 0; -moz-box-shadow: 0 0 0; box-shadow: 0 0 0; }
.nav-bar > li:last-child { border-right: solid 1px #333333; -webkit-box-shadow: 1px 0 0 rgba(255, 255, 255, 0.2) inset, 1px 0 0 rgba(255, 255, 255, 0.2); -moz-box-shadow: 1px 0 0 rgba(255, 255, 255, 0.2) inset, 1px 0 0 rgba(255, 255, 255, 0.2); box-shadow: 1px 0 0 rgba(255, 255, 255, 0.2) inset, 1px 0 0 rgba(255, 255, 255, 0.2); }
.nav-bar > li.active { background: #2ba6cb; border-color: #2284a1; }
.nav-bar > li.active > a { color: white; cursor: default; }
.nav-bar > li.active:hover { background: #2ba6cb; cursor: default; }
.nav-bar > li:hover { background: #333333; }
.nav-bar > li a { color: #e6e6e6; }
.nav-bar > li ul { margin-bottom: 0; }
.nav-bar > li .flyout { display: none; }
.nav-bar > li.has-flyout > a:first-child { padding-right: 36px; position: relative; }
.nav-bar > li.has-flyout > a:first-child:after { content: ""; display: block; width: 0; height: 0; border: solid 4px; border-color: #e6e6e6 transparent transparent transparent; position: absolute; right: 20px; top: 17px; }
.nav-bar > li.has-flyout > a.flyout-toggle { border-left: 0 !important; position: absolute; right: 0; top: 0; padding: 22px; z-index: 2; display: block; }
.nav-bar > li.has-flyout.is-touch > a:first-child { padding-right: 55px; }
.nav-bar > li.has-flyout.is-touch > a.flyout-toggle { border-left: 1px dashed #666; }
.nav-bar > li > a:first-child { position: relative; padding: 0 20px; display: block; text-decoration: none; font-size: 14px; }
.nav-bar > li > input { margin: 0 10px; }
.nav-bar.vertical { height: auto; margin-top: 0; }
.nav-bar.vertical > li { float: none; border-bottom: none; }
.nav-bar.vertical > li.has-flyout > a:first-child:after { content: ""; display: block; width: 0; height: 0; border: solid 4px; border-color: transparent transparent transparent #e6e6e6; }
.nav-bar.vertical > li .flyout { left: 100%; top: -1px; }
.nav-bar.vertical > li .flyout.right { left: auto; right: 100%; }

.flyout { background: #f2f2f2; padding: 20px; margin: 0; border: 1px solid #d9d9d9; position: absolute; top: 39px; left: -1px; width: 250px; z-index: 40; -webkit-box-shadow: 0 1px 5px rgba(0, 0, 0, 0.1); -moz-box-shadow: 0 1px 5px rgba(0, 0, 0, 0.1); box-shadow: 0 1px 5px rgba(0, 0, 0, 0.1); /* remove margin on any first-child element */ /* remove margin on last element */ }
.flyout p { line-height: 1.2; font-size: 13px; }
.flyout *:first-child { margin-top: 0; }
.flyout *:last-child { margin-bottom: 0; }
.flyout.small { width: 166.667px; }
.flyout.large { width: 437.5px; }
.flyout.right { left: auto; right: -2px; }
.flyout.up { top: auto; bottom: 39px; }

ul.flyout, .nav-bar li ul { padding: 0; list-style: none; }
ul.flyout li, .nav-bar li ul li { border-left: solid 3px #CCC; }
ul.flyout li a, .nav-bar li ul li a { background: #f2f2f2; border: 1px solid #e6e6e6; border-width: 1px 1px 0 0; color: #555; display: block; font-size: 14px; height: auto; line-height: 1; padding: 15px 20px; -webkit-box-shadow: 0 1px 0 rgba(255, 255, 255, 0.5) inset; -moz-box-shadow: 0 1px 0 rgba(255, 255, 255, 0.5) inset; box-shadow: 0 1px 0 rgba(255, 255, 255, 0.5) inset; }
ul.flyout li a:hover, .nav-bar li ul li a:hover { background: #ebebeb; color: #333; }
ul.flyout li.active, .nav-bar li ul li.active { margin-top: 0; border-top: 1px solid #4d4d4d; border-left: 4px solid #1a1a1a; }
ul.flyout li.active a, .nav-bar li ul li.active a { background: #4d4d4d; border: none; color: #fff; height: auto; margin: 0; position: static; top: 0; -webkit-box-shadow: 0 0 0; -moz-box-shadow: 0 0 0; box-shadow: 0 0 0; }

/* Mobile Styles */
@media only screen and (max-device-width: 1280px) { .touch .nav-bar li.has-flyout > a { padding-right: 36px !important; } }
@media only screen and (max-width: 1279px) and (min-width: 768px) { .touch .nav-bar li a { font-size: 13px; font-size: 1.3rem; }
  .touch .nav-bar li.has-flyout > a.flyout-toggle { padding: 20px !important; }
  .touch .nav-bar li.has-flyout > a { padding-right: 36px !important; } }
@media only screen and (max-width: 767px) { .nav-bar { height: auto; }
  .nav-bar > li { float: none; display: block; border-right: none; }
  .nav-bar > li > a.main { text-align: left; border-top: 1px solid #ddd; border-right: none; }
  .nav-bar > li:first-child > a.main { border-top: none; }
  .nav-bar > li.has-flyout > a.flyout-toggle { position: absolute; right: 0; top: 0; padding: 22px; z-index: 2; display: block; }
  .nav-bar > li.has-flyout.is-touch > a.flyout-toggle span { content: ""; width: 0; height: 0; display: block; }
  .nav-bar > li.has-flyout > a.flyout-toggle:hover span { border-top-color: #141414; }
  .nav-bar.vertical > li.has-flyout > .flyout { left: 0; }
  .flyout { position: relative; width: 100% !important; top: auto; margin-right: -2px; border-width: 1px 1px 0 1px; }
  .flyout.right { float: none; right: auto; left: -1px; }
  .flyout.small, .flyout.large { width: 100% !important; }
  .flyout p:last-child { margin-bottom: 18px; } }

/* Requires globals.css */
/* Standard Forms ---------------------- */
form { margin: 0 0 19.416px; }

.row form .row { margin: 0 -6px; }
.row form .row .column, .row form .row .columns { padding: 0 6px; }
.row form .row.collapse { margin: 0; }
.row form .row.collapse .column, .row form .row.collapse .columns { padding: 0; }

label { font-size: 14px; color: #4d4d4d; cursor: pointer; display: block; font-weight: 500; margin-bottom: 3px; }
label.right { float: none; text-align: right; }
label.inline { line-height: 32px; margin: 0 0 12px 0; }

@media only screen and (max-width: 767px) { label.right { text-align: left; } }
.prefix, .postfix { display: block; position: relative; z-index: 2; text-align: center; width: 100%; padding-top: 0; padding-bottom: 0; height: 32px; line-height: 31px; }

a.button.prefix, a.button.postfix { padding-left: 0; padding-right: 0; text-align: center; }

span.prefix, span.postfix { background: #f2f2f2; border: 1px solid #cccccc; }

.prefix { left: 2px; -moz-border-radius-topleft: 2px; -webkit-border-top-left-radius: 2px; border-top-left-radius: 2px; -moz-border-radius-bottomleft: 2px; -webkit-border-bottom-left-radius: 2px; border-bottom-left-radius: 2px; }

.postfix { right: 2px; -moz-border-radius-topright: 2px; -webkit-border-top-right-radius: 2px; border-top-right-radius: 2px; -moz-border-radius-bottomright: 2px; -webkit-border-bottom-right-radius: 2px; border-bottom-right-radius: 2px; }

input[type="text"], input[type="password"], input[type="date"], input[type="datetime"], input[type="email"], input[type="number"], input[type="search"], input[type="tel"], input[type="time"], input[type="url"], textarea { border: 1px solid #cccccc; -webkit-border-radius: 2px; -moz-border-radius: 2px; -ms-border-radius: 2px; -o-border-radius: 2px; border-radius: 2px; -webkit-box-shadow: inset 0 1px 2px rgba(0, 0, 0, 0.1); -moz-box-shadow: inset 0 1px 2px rgba(0, 0, 0, 0.1); box-shadow: inset 0 1px 2px rgba(0, 0, 0, 0.1); color: rgba(0, 0, 0, 0.75); display: block; font-size: 14px; margin: 0 0 12px 0; padding: 6px; height: 32px; width: 100%; -webkit-transition: all 0.15s linear; -moz-transition: all 0.15s linear; -o-transition: all 0.15s linear; transition: all 0.15s linear; }
input[type="text"].oversize, input[type="password"].oversize, input[type="date"].oversize, input[type="datetime"].oversize, input[type="email"].oversize, input[type="number"].oversize, input[type="search"].oversize, input[type="tel"].oversize, input[type="time"].oversize, input[type="url"].oversize, textarea.oversize { font-size: 18px !important; font-size: 1.8rem !important; }
input[type="text"]:focus, input[type="password"]:focus, input[type="date"]:focus, input[type="datetime"]:focus, input[type="email"]:focus, input[type="number"]:focus, input[type="search"]:focus, input[type="tel"]:focus, input[type="time"]:focus, input[type="url"]:focus, textarea:focus { background: #fafafa; outline: none !important; border-color: #b3b3b3; }
input[type="text"][disabled], input[type="password"][disabled], input[type="date"][disabled], input[type="datetime"][disabled], input[type="email"][disabled], input[type="number"][disabled], input[type="search"][disabled], input[type="tel"][disabled], input[type="time"][disabled], input[type="url"][disabled], textarea[disabled] { background-color: #ddd; }

textarea { height: auto; }

select { width: 100%; }

/* Fieldsets */
fieldset { border: solid 1px #ddd; border-radius: 3px; -webkit-border-radius: 3px; -moz-border-radius: 3px; padding: 12px 12px 0; margin: 18px 0; }
fieldset legend { font-weight: bold; background: white; padding: 0 3px; margin: 0 0 0 -3px; }

/* Errors */
.error input, input.error { border-color: #c60f13; background-color: rgba(198, 15, 19, 0.1); }

.error label, label.error { color: #c60f13; }

.error small, small.error { display: block; padding: 6px 4px; margin-top: -13px; margin-bottom: 12px; background: #c60f13; color: #fff; font-size: 12px; font-size: 1.2rem; font-weight: bold; -moz-border-radius-bottomleft: 2px; -webkit-border-bottom-left-radius: 2px; border-bottom-left-radius: 2px; -moz-border-radius-bottomright: 2px; -webkit-border-bottom-right-radius: 2px; border-bottom-right-radius: 2px; }

@media only screen and (max-width: 767px) { input[type="text"].one, textarea.one { width: 100% !important; }
  input[type="text"].two, textarea.two { width: 100% !important; }
  input[type="text"].three, textarea.three { width: 100% !important; }
  input[type="text"].four, textarea.four { width: 100% !important; }
  input[type="text"].five, textarea.five { width: 100% !important; }
  input[type="text"].six, textarea.six { width: 100% !important; }
  input[type="text"].seven, textarea.seven { width: 100% !important; }
  input[type="text"].eight, textarea.eight { width: 100% !important; }
  input[type="text"].nine, textarea.nine { width: 100% !important; }
  input[type="text"].ten, textarea.ten { width: 100% !important; }
  input[type="text"].eleven, textarea.eleven { width: 100% !important; }
  input[type="text"].twelve, textarea.twelve { width: 100% !important; } }
/* Custom Forms ---------------------- */
form.custom { /* Custom input, disabled */ }
form.custom span.custom { display: inline-block; width: 16px; height: 16px; position: relative; top: 2px; border: solid 1px #ccc; background: #fff; }
form.custom span.custom.radio { -webkit-border-radius: 100px; -moz-border-radius: 100px; -ms-border-radius: 100px; -o-border-radius: 100px; border-radius: 100px; }
form.custom span.custom.checkbox:before { content: "\00d7"; display: block; line-height: 0.8; height: 14px; width: 14px; text-align: center; position: absolute; top: 0; left: 0; /* margin-top: -9px; margin-left: -4px; */ font-size: 14px; color: #fff; }
form.custom span.custom.radio.checked:before { content: ""; display: block; width: 8px; height: 8px; -webkit-border-radius: 100px; -moz-border-radius: 100px; -ms-border-radius: 100px; -o-border-radius: 100px; border-radius: 100px; background: #222; position: relative; top: 3px; left: 3px; }
form.custom span.custom.checkbox.checked:before { color: #222; }
form.custom div.custom.dropdown { display: block; position: relative; width: auto; height: 28px; margin-bottom: 9px; margin-top: 2px; }
form.custom div.custom.dropdown a.current { display: block; width: auto; line-height: 26px; min-height: 28px; padding: 0 38px 0 6px; border: solid 1px #ddd; color: #141414; background-color: #fff; white-space: nowrap; }
form.custom div.custom.dropdown a.selector { position: absolute; width: 27px; height: 28px; display: block; right: 0; top: 0; border: solid 1px #ddd; }
form.custom div.custom.dropdown a.selector:after { content: ""; display: block; content: ""; display: block; width: 0; height: 0; border: solid 5px; border-color: #aaaaaa transparent transparent transparent; position: absolute; left: 50%; top: 50%; margin-top: -2px; margin-left: -5px; }
form.custom div.custom.dropdown:hover a.selector:after, form.custom div.custom.dropdown.open a.selector:after { content: ""; display: block; width: 0; height: 0; border: solid 5px; border-color: #222222 transparent transparent transparent; }
form.custom div.custom.dropdown.open ul { display: block; z-index: 10; }
form.custom div.custom.dropdown.small { width: 134px !important; }
form.custom div.custom.dropdown.medium { width: 254px !important; }
form.custom div.custom.dropdown.large { width: 434px !important; }
form.custom div.custom.dropdown.expand { width: 100% !important; }
form.custom div.custom.dropdown.open.small ul { width: 134px !important; }
form.custom div.custom.dropdown.open.medium ul { width: 254px !important; }
form.custom div.custom.dropdown.open.large ul { width: 434px !important; }
form.custom div.custom.dropdown.open.expand ul { width: 100% !important; }
form.custom div.custom.dropdown ul { position: absolute; width: auto; display: none; margin: 0; left: 0; top: 27px; margin: 0; padding: 0; background: #fff; background: rgba(255, 255, 255, 0.95); border: solid 1px #cccccc; }
form.custom div.custom.dropdown ul li { color: #555; font-size: 13px; cursor: pointer; padding: 3px 38px 3px 6px; min-height: 18px; line-height: 18px; margin: 0; white-space: nowrap; list-style: none; }
form.custom div.custom.dropdown ul li.selected { background: #cdebf5; color: #000; }
form.custom div.custom.dropdown ul li.selected:after { content: "\2013"; position: absolute; right: 10px; }
form.custom div.custom.dropdown ul li:hover { background-color: #e3f4f9; color: #222; }
form.custom div.custom.dropdown ul li:hover:after { content: "\2013"; position: absolute; right: 10px; color: #8ed3e7; }
form.custom div.custom.dropdown ul li.selected:hover { background: #cdebf5; cursor: default; color: #000; }
form.custom div.custom.dropdown ul li.selected:hover:after { color: #000; }
form.custom div.custom.dropdown ul.show { display: block; }
form.custom .custom.disabled { background-color: #ddd; }

/* Correct FF custom dropdown height */
@-moz-document url-prefix() { form.custom div.custom.dropdown a.selector { height: 30px; } }

.lt-ie9 form.custom div.custom.dropdown a.selector { height: 30px; }

/* CSS for jQuery Orbit Plugin 1.4.0 Maintained for Foundation. foundation.zurb.com Free to use under the MIT license. http://www.opensource.org/licenses/mit-license.php
*/
/* Container ---------------------- */
div.orbit-wrapper { width: 1px; height: 1px; position: relative; }

div.orbit { width: 1px; height: 1px; position: relative; overflow: hidden; margin-bottom: 17px; }

div.orbit.with-bullets { margin-bottom: 40px; }

div.orbit .orbit-slide { max-width: 100%; position: absolute; top: 0; left: 0; }

div.orbit a.orbit-slide { border: none; line-height: 0; display: none; }

div.orbit div.orbit-slide { width: 100%; height: 100%; }

/* Note: If your slider only uses content or anchors, you're going to want to put the width and height declarations on the ".orbit>div" and "div.orbit>a" tags in addition to just the .orbit-wrapper */
/* Timer ---------------------- */
div.orbit-wrapper div.timer { width: 40px; height: 40px; overflow: hidden; position: absolute; top: 10px; right: 10px; opacity: .6; cursor: pointer; z-index: 31; }

div.orbit-wrapper span.rotator { display: block; width: 40px; height: 40px; position: absolute; top: 0; left: -20px; background: url('/images/foundation/orbit/rotator-black.png?1341958611') no-repeat; z-index: 3; }

div.orbit-wrapper span.mask { display: block; width: 20px; height: 40px; position: absolute; top: 0; right: 0; z-index: 2; overflow: hidden; }

div.orbit-wrapper span.rotator.move { left: 0; }

div.orbit-wrapper span.mask.move { width: 40px; left: 0; background: url('/images/foundation/orbit/timer-black.png?1341958611') repeat 0 0; }

div.orbit-wrapper span.pause { display: block; width: 40px; height: 40px; position: absolute; top: 0; left: 0; background: url('/images/foundation/orbit/pause-black.png?1341958611') no-repeat; z-index: 4; opacity: 0; }

div.orbit-wrapper span.pause.active { background: url('/images/foundation/orbit/pause-black.png?1341958611') no-repeat 0 -40px; }

div.orbit-wrapper div.timer:hover span.pause, div.orbit-wrapper span.pause.active { opacity: 1; }

/* Captions ---------------------- */
.orbit-caption { display: none; font-family: "HelveticaNeue", "Helvetica-Neue", Helvetica, Arial, sans-serif; }

.orbit-wrapper .orbit-caption { background: #000; background: rgba(0, 0, 0, 0.6); z-index: 30; color: #fff; text-align: center; padding: 7px 0; font-size: 13px; font-size: 1.3rem; position: absolute; right: 0; bottom: 0; width: 100%; }

/* Directional Nav ---------------------- */
div.orbit-wrapper div.slider-nav { display: block; }

div.orbit-wrapper div.slider-nav span { width: 39px; height: 50px; text-indent: -9999px; position: absolute; z-index: 30; top: 50%; margin-top: -25px; cursor: pointer; }

div.orbit-wrapper div.slider-nav span.right { background: url('/images/foundation/orbit/right-arrow.png?1341958611'); background-size: 100%; right: 0; }

div.orbit-wrapper div.slider-nav span.left { background: url('/images/foundation/orbit/left-arrow.png?1341958611'); background-size: 100%; left: 0; }

.lt-ie9 div.orbit-wrapper div.slider-nav span.right { background: url('/images/foundation/orbit/right-arrow-small.png?1341958611'); }
.lt-ie9 div.orbit-wrapper div.slider-nav span.left { background: url('/images/foundation/orbit/left-arrow-small.png?1341958611'); }

/* Bullet Nav ---------------------- */
ul.orbit-bullets { position: absolute; z-index: 30; list-style: none; bottom: -40px; left: 50%; margin-left: -50px; padding: 0; }

ul.orbit-bullets li { float: left; margin-left: 5px; cursor: pointer; color: #999; text-indent: -9999px; background: url('/images/foundation/orbit/bullets.jpg?1341958611') no-repeat 4px 0; width: 13px; height: 12px; overflow: hidden; }

ul.orbit-bullets li.active { color: #222; background-position: -8px 0; }

ul.orbit-bullets li.has-thumb { background: none; width: 100px; height: 75px; }

ul.orbit-bullets li.active.has-thumb { background-position: 0 0; border-top: 2px solid #000; }

/* Fluid Layout ---------------------- */
div.orbit img.fluid-placeholder { visibility: hidden; position: static; display: block; width: 100%; }

div.orbit, div.orbit-wrapper { width: 100% !important; }

ul.orbit-bullets { position: absolute; z-index: 30; list-style: none; bottom: -50px; left: 50%; margin-left: -50px; padding: 0; }

ul.orbit-bullets li { float: left; margin-left: 5px; cursor: pointer; color: #999; text-indent: -9999px; background: url('/images/foundation/orbit/bullets.jpg?1341958611') no-repeat 4px 0; width: 13px; height: 12px; overflow: hidden; }

ul.orbit-bullets li.has-thumb { background: none; width: 100px; height: 75px; }

ul.orbit-bullets li.active { color: #222; background-position: -8px 0; }

ul.orbit-bullets li.active.has-thumb { background-position: 0 0; border-top: 2px solid #000; }

/* Correct timer in IE */
.lt-ie9 .timer { display: none !important; }

.lt-ie9 div.caption { background: transparent; filter: progid:DXImageTransform.Microsoft.gradient(startColorstr=#99000000,endColorstr=#99000000); zoom: 1; }

/* CSS for jQuery Reveal Plugin Maintained for Foundation. foundation.zurb.com Free to use under the MIT license. http://www.opensource.org/licenses/mit-license.php */
/* Reveal Modals ---------------------- */
.reveal-modal-bg { position: fixed; height: 100%; width: 100%; background: #000; background: rgba(0, 0, 0, 0.45); z-index: 40; display: none; top: 0; left: 0; }

.reveal-modal { background: white; visibility: hidden; display: none; top: 100px; left: 50%; margin-left: -260px; width: 520px; position: absolute; z-index: 41; padding: 30px; -webkit-box-shadow: 0 0 10px rgba(0, 0, 0, 0.4); -moz-box-shadow: 0 0 10px rgba(0, 0, 0, 0.4); box-shadow: 0 0 10px rgba(0, 0, 0, 0.4); }
.reveal-modal *:first-child { margin-top: 0; }
.reveal-modal *:last-child { margin-bottom: 0; }
.reveal-modal .close-reveal-modal { font-size: 22px; font-size: 2.2rem; line-height: .5; position: absolute; top: 8px; right: 11px; color: #aaa; text-shadow: 0 -1px 1px rbga(0, 0, 0, 0.6); font-weight: bold; cursor: pointer; }
.reveal-modal.small { width: 30%; margin-left: -10%; }
.reveal-modal.medium { width: 40%; margin-left: -20%; }
.reveal-modal.large { width: 60%; margin-left: -30%; }
.reveal-modal.expand { width: 90%; margin-left: -45%; }
.reveal-modal .row { min-width: 0; }

/* Mobile */
@media only screen and (max-width: 767px) { .reveal-modal-bg { position: absolute; }
  .reveal-modal, .reveal-modal.small, .reveal-modal.medium, .reveal-modal.large, .reveal-modal.xlarge { width: 80%; top: 15px; left: 50%; margin-left: -40%; padding: 20px; height: auto; } }
  /* NOTES Close button entity is &#215;
 Example markup <div id="myModal" class="reveal-modal"> <h2>Awesome. I have it.</h2> <p class="lead">Your couch.  I it's mine.</p> <p>Lorem ipsum dolor sit amet, consectetur adipiscing elit. In ultrices aliquet placerat. Duis pulvinar orci et nisi euismod vitae tempus lorem consectetur. Duis at magna quis turpis mattis venenatis eget id diam. </p> <a class="close-reveal-modal">&#215;</a> </div> */
