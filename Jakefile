var  jake = require("jake");
var app = {};
var db = require('./backend/db.js')(app);
var config = require('./config.json')

task('default', [], function () {
  console.log('This is the default task.');
});

namespace('traction', function () {
	task('install', ["db:init"], function () {	  
		console.log(">> running traction:install");
	})

	task('update', ["db:update"], function () {	  
		console.log(">> running traction:update");

	})

	namespace('db', function () {
		task('update', ["migrate"], function () {	  
			console.log(">> running traction:db:update");
		})

		task('init', [], function () {	  
			console.log(">> running traction:db:init");
			var masterUser;
			return app.db.connection.sync({force: true}).then(function() {
				// Create view for mosquitto auth
				return app.db.connection.query("CREATE OR REPLACE VIEW Auth AS SELECT u.id, u.username, d.devicename, d.accessToken FROM Users AS u JOIN Devices AS d WHERE u.id = d.userId;")
			}).then(function(){
				return app.db.connection.query("CREATE OR REPLACE VIEW Access AS SELECT p.username, p.topic, p.rw, p.userId, p.deviceId, p.shareId FROM Permissions AS p LEFT JOIN Shares AS s ON p.shareId = s.id WHERE s.accepted IS NULL OR s.accepted = 1");
			}).then(function() {
				// Sequelize creates a unique index for trackedUserId and trackingUserId
				// There might however be entries with duplicate trackedUserId and trackingUserId if a user has more than one device 
				// The Share model defines a new index that also includes the trackedDeviceId in an unique index so the Shares_trackedUserId_trackingUserId_unique one is redundant and wrong
				// It can safely be dropped as the new index includes all components in the same order to maintain foreign key integrity
				console.log("Fixing indices of Shares table");
				return app.db.connection.query("DROP INDEX `Shares_trackedUserId_trackingUserId_unique` on `Shares`");
			}).then(function() {
			        	return app.db.models.User.create({
       					         username : config.broker.user.split("|")[0],
               					 email : "undefined@example.org",
               					 password : config.broker.password
        				})

				}).then(function(user))
				return app.db.models.Permission.create({
					username : user.username,
					topic : "#",
					rw: "2"
				})
			})


			.catch(function(error) { 
				console.error("error: " + error); 
			});
		})

	})
})
