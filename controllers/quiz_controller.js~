var models = require("../models");
var Sequelize = require('sequelize');

var paginate = require('../helpers/paginate').paginate;

// Autoload el quiz asociado a :quizId
exports.load = function (req, res, next, quizId) {

    models.Quiz.findById(quizId)
    .then(function (quiz) {
        if (quiz) {
            req.quiz = quiz;
            next();
        } else {
            throw new Error('No existe ningún quiz con id=' + quizId);
        }
    })
    .catch(function (error) {
        next(error);
    });
};


// GET /quizzes
exports.index = function (req, res, next) {

    var countOptions = {};

    // Busquedas:
    var search = req.query.search || '';
    if (search) {
        var search_like = "%" + search.replace(/ +/g,"%") + "%";

        countOptions.where = {question: { $like: search_like }};
    }

    models.Quiz.count(countOptions)
    .then(function (count) {

        // Paginacion:

        var items_per_page = 10;

        // La pagina a mostrar viene en la query
        var pageno = parseInt(req.query.pageno) || 1;

        // Crear un string con el HTML que pinta la botonera de paginacion.
        // Lo añado como una variable local de res para que lo pinte el layout de la aplicacion.
        res.locals.paginate_control = paginate(count, items_per_page, pageno, req.url);

        var findOptions = countOptions;

        findOptions.offset = items_per_page * (pageno - 1);
        findOptions.limit = items_per_page;

        return models.Quiz.findAll(findOptions);
    })
    .then(function (quizzes) {
        res.render('quizzes/index.ejs', {
            quizzes: quizzes,
            search: search
        });
    })
    .catch(function (error) {
        next(error);
    });
};


// GET /quizzes/:quizId
exports.show = function (req, res, next) {

    res.render('quizzes/show', {quiz: req.quiz});
};


// GET /quizzes/new
exports.new = function (req, res, next) {

    var quiz = {question: "", answer: ""};

    res.render('quizzes/new', {quiz: quiz});
};


// POST /quizzes/create
exports.create = function (req, res, next) {

    var quiz = models.Quiz.build({
        question: req.body.question,
        answer: req.body.answer
    });

    // guarda en DB los campos pregunta y respuesta de quiz
    quiz.save({fields: ["question", "answer"]})
    .then(function (quiz) {
        req.flash('success', 'Quiz creado con éxito.');
        res.redirect('/quizzes/' + quiz.id);
    })
    .catch(Sequelize.ValidationError, function (error) {

        req.flash('error', 'Errores en el formulario:');
        for (var i in error.errors) {
            req.flash('error', error.errors[i].value);
        }

        res.render('quizzes/new', {quiz: quiz});
    })
    .catch(function (error) {
        req.flash('error', 'Error al crear un Quiz: ' + error.message);
        next(error);
    });
};


// GET /quizzes/:quizId/edit
exports.edit = function (req, res, next) {

    res.render('quizzes/edit', {quiz: req.quiz});
};


// PUT /quizzes/:quizId
exports.update = function (req, res, next) {

    req.quiz.question = req.body.question;
    req.quiz.answer = req.body.answer;

    req.quiz.save({fields: ["question", "answer"]})
    .then(function (quiz) {
        req.flash('success', 'Quiz editado con éxito.');
        res.redirect('/quizzes/' + req.quiz.id);
    })
    .catch(Sequelize.ValidationError, function (error) {

        req.flash('error', 'Errores en el formulario:');
        for (var i in error.errors) {
            req.flash('error', error.errors[i].value);
        }

        res.render('quizzes/edit', {quiz: req.quiz});
    })
    .catch(function (error) {
        req.flash('error', 'Error al editar el Quiz: ' + error.message);
        next(error);
    });
};


// DELETE /quizzes/:quizId
exports.destroy = function (req, res, next) {

    req.quiz.destroy()
    .then(function () {
        req.flash('success', 'Quiz borrado con éxito.');
        res.redirect('/quizzes');
    })
    .catch(function (error) {
        req.flash('error', 'Error al editar el Quiz: ' + error.message);
        next(error);
    });
};


// GET /quizzes/:quizId/play
exports.play = function (req, res, next) {

    var answer = req.query.answer || '';

    res.render('quizzes/play', {
        quiz: req.quiz,
        answer: answer
    });
};


// GET /quizzes/:quizId/check
exports.check = function (req, res, next) {

    var answer = req.query.answer || "";

    var result = answer.toLowerCase().trim() === req.quiz.answer.toLowerCase().trim();

    res.render('quizzes/result', {
        quiz: req.quiz,
        result: result,
        answer: answer
    });
};

//RAMDOMPLAY
exports.randomplay = function(req, res, next){

// intentar leer score de la sesión y si no existe, lo creo poniéndolo a cero

if(!req.session.score){
 req.session.score = 0;
}
else{
req.session.score = req.session.score;
}


// creo el array de preguntascontestadas también en la sesión
// si existe, es que ya está iniciada la partida, así que leo el array preguntas contestadas

if(!req.session.preguntasContestadas){
 req.session.preguntasContestadas = [];
} 

if(req.session.partidaTerminada){
	req.session.partidaTerminada = false;
	req.session.score = 0;
	req.session.preguntasContestadas=[];
}


models.Quiz.count().then(function(count){

	var numPregunta = Math.floor((Math.random()*count)) + 1;
	// bucle while que haga: mientras que numPregunta esté en preguntasContestadas, calcularnumPregunta 
while(req.session.preguntasContestadas.indexOf(numPregunta)!=-1){

	var numPregunta = Math.floor((Math.random()*count)) + 1;
}
// cuando ya sabes numPregunta, añadirlo al array de preguntasContestadas
	req.session.preguntasContestadas.push(numPregunta);
	console.log("Este es el número" + numPregunta);
// obtener el Quiz de la base de datos que tiene el ID numPregunta
	models.Quiz.findById(numPregunta).then(function(quiz){
// enviar la vista randomplay(quiz, score)
		if(quiz){
			res.render('quizzes/random_play',{
				score: req.session.score,
				quiz: quiz
			});
			} else{
			res.send("ERROR");
			}
		});

	});
}


//RANDOM CHECK
exports.randomcheck = function (req, res, next) {

    models.Quiz.count().then(function(count){

	    var answer = req.query.answer || ""; // Recibe respuesta en parámetro answer de query y
		                                 // si no existe, lo inicializa con "". 
		                                 // (Se puede enviar respuesta en blanco).

	 // Comprueba si la respuesta es correcta.    
	    var result = answer.toLowerCase().trim() === req.quiz.answer.toLowerCase().trim(); 

	 // Si la respuesta es correcta sumar +1 a score
	    if (result) {
		req.session.score = req.session.score + 1;
	    } else {
		req.session.score = 0;
	    }


	 // Si todavía tengo preguntas por contestar
	    if ( req.session.preguntasContestadas.length !== count) { 

		if (result) {
			req.session.partidaTerminada = false;
	    	} else { 
			req.session.partidaTerminada = true;
		}
	   // si quedan preguntas, enviar result
	    	res.render('quizzes/random_result', {  // Página que indica si se contestó bien o no
			result: result,
			answer: answer,
			score: req.session.score
	    	});
	   

	    } else { // si no quedan preguntas, enviar none

		req.session.partidaTerminada = true;

	    	res.render('quizzes/random_none', {  // Página que indica que no hay más preguntas 
			score: req.session.score       // (se han contestado a todas las preguntas
                                                       // correctamente)
	    	});	
	    }	
    });
}




