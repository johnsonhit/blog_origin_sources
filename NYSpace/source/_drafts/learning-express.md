mysql[root:196416@nY]
CREATE USER 'test'@'localhost' IDENTIFIED BY 'test';
GRANT ALL PRIVILEGES ON *.* TO 'test'@'localhost'
WITH GRANT OPTION;

CREATE TABLE IF NOT EXISTS `user` (
	`id` int(11) NOT NULL AUTO_INCREMENT, 
	`name` varchar(50), 
	`age` tinyint(4),
	PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 DEFAULT COLLATE=utf8_unicode_ci;

INSERT INTO `user` (`name`, `age`) VALUES ("niyaoyao", 18);