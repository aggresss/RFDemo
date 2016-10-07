# build helloworld executable when user executes "make"  
  
helloworld: helloworld.o  
	$(CC) $(LDFLAGS) helloworld.o -o helloworld  
  
helloworld.o: helloworld.c  
	$(CC) $(CFLAGS) -c helloworld.c  
  
# remove object files and executable when user executes "make clean"  
clean: 
	rm *.o helloworld  
