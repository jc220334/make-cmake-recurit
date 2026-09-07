CC = gcc
CFLAGS = -Wall -Wextra -Iinclude

calculator: main.o calculator.o logger.o
	gcc $(CFLAGS) -o calculator main.o calculator.o logger.o
	# TODO: link object files into the calculator executable

main.o: src/main.c include/calculator.h include/logger.h
	gcc $(CFLAGS) -c src/main.c -o main.o
	# TODO: compile src/main.c

calculator.o: src/calculator.c include/calculator.h
	gcc $(CFLAGS) -c src/calculator.c -o calculator.o
	# TODO: compile src/calculator.c

logger.o: src/logger.c include/logger.h
	gcc $(CFLAGS) -c src/logger.c -o logger.o
	# TODO: compile src/logger.c

.PHONY: clean
clean:
	rm -f calculator main.o calculator.o logger.o
	# TODO: remove calculator and all .o files
