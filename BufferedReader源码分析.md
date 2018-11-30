# BufferedReader源码分析  
由于经常使用BufferedReader的readLine方法，有一些疑惑，所以今天来看看BufferedReader源码是如何实现的  

### 一、属性介绍  
BufferedReader中定义了如下属性：  

	public class BufferedReader extends Reader {  
		private Reader in;		//真正从IO流中读取信息的Reader

	    private char cb[];		//缓冲区，IO流中的东西被in读到这个数组里面
	    private int nChars, nextChar;		//nextChar作为下一次readLine调用的时候，从cb缓冲区   										中开始读的下标
	
	    private static final int INVALIDATED = -2;
	    private static final int UNMARKED = -1;
	    private int markedChar = UNMARKED;		//不知所云，在readLine方法中没有涉及
	    private int readAheadLimit = 0; 		//不知所云，在readLine方法中没有涉及
	
	    private boolean skipLF = false;			//如果在一次readLine的调用中，因为遇到了'\r'导致了return，该值变为true，且如果这时候cb[nextChar]是'\n'是nextChar执行加一操作
	
	    /** The skipLF flag when the mark was set */
	    private boolean markedSkipLF = false;		//不知所云，在readLine方法中没有涉及
	
	    private static int defaultCharBufferSize = 8192;     //cb缓冲区默认大小
	    private static int defaultExpectedLineLength = 80;		//不知所云，在readLine方法中没有涉及
	}  

### 二、构造器  
BufferedReader提供了两种构造器：  

	public BufferedReader(Reader in, int sz) {
        super(in);
        if (sz <= 0)
            throw new IllegalArgumentException("Buffer size <= 0");
        this.in = in;
        cb = new char[sz];
        nextChar = nChars = 0;
    }  

	public BufferedReader(Reader in) {
        this(in, defaultCharBufferSize);
    }  

可见，我们常用的方式只是第二种，那么其缓冲数组的大小按照默认提供的大小。  

值得注意的是构造器为**nextChar、nChars都初始化为了0**  

### 三、相关方法  

	//检查in这个reader
	private void ensureOpen() throws IOException {
        if (in == null)
            throw new IOException("Stream closed");
    }

	//负责填充缓冲数组cb
	private void fill() throws IOException {
        int dst;
        if (markedChar <= UNMARKED) {
            /* No mark */
            dst = 0;
        } else {
            /* Marked */
            int delta = nextChar - markedChar;
            if (delta >= readAheadLimit) {
                /* Gone past read-ahead limit: Invalidate mark */
                markedChar = INVALIDATED;
                readAheadLimit = 0;
                dst = 0;
            } else {
                if (readAheadLimit <= cb.length) {
                    /* Shuffle in the current buffer */
                    System.arraycopy(cb, markedChar, cb, 0, delta);
                    markedChar = 0;
                    dst = delta;
                } else {
                    /* Reallocate buffer to accommodate read-ahead limit */
                    char ncb[] = new char[readAheadLimit];
                    System.arraycopy(cb, markedChar, ncb, 0, delta);
                    cb = ncb;
                    markedChar = 0;
                    dst = delta;
                }
                nextChar = nChars = delta;
            }
        }

        int n;
        do {
            n = in.read(cb, dst, cb.length - dst);
        } while (n == 0);
        if (n > 0) {
            nChars = dst + n;
            nextChar = dst;
        }
    }

### 四、readLine方法实现  
readLine从名字上来看是读取一行的意思，但是一行的界定是什么经常会疑惑，这就来看看是如何实现的：  

	String readLine(boolean ignoreLF) throws IOException {
        StringBuffer s = null;
        int startChar;

        synchronized (lock) {
			//检查in
            ensureOpen();
            boolean omitLF = ignoreLF || skipLF;

        bufferLoop:
            for (;;) {

				//----------1号----------
                if (nextChar >= nChars)
                    fill();

				//----------2号----------
                if (nextChar >= nChars) { /* EOF */
                    if (s != null && s.length() > 0)
                        return s.toString();
                    else
                        return null;
                }
                boolean eol = false;
                char c = 0;
                int i;

                /* Skip a leftover '\n', if necessary */
				//----------3号----------
                if (omitLF && (cb[nextChar] == '\n'))
                    nextChar++;
                skipLF = false;
                omitLF = false;

            charLoop:
                for (i = nextChar; i < nChars; i++) {
                    c = cb[i];
					//-----------4号----------
                    if ((c == '\n') || (c == '\r')) {
                        eol = true;
                        break charLoop;
                    }
                }

                startChar = nextChar;
                nextChar = i;

				//-----------5号----------
                if (eol) {
                    String str;
					//----------6号-----------
                    if (s == null) {
                        str = new String(cb, startChar, i - startChar);
                    } else {
                        s.append(cb, startChar, i - startChar);
                        str = s.toString();
                    }
                    nextChar++;
					//-----------7号-----------
                    if (c == '\r') {
                        skipLF = true;
                    }
                    return str;
                }
				//----------8号----------
                if (s == null)
                    s = new StringBuffer(defaultExpectedLineLength);
                s.append(cb, startChar, i - startChar);
            }
        }
    }  

这个方法中的参数是怎么一回事？我们平常不是这么调用的啊？  

	public String readLine() throws IOException {
        return readLine(false);
    }  

我们平时的调用方法，默认传入的参数为false。  

首先，我们先看一看在第一次调用readLine方法时，各种局部变量的值：首先，由于ignoreLF传入时为false，而且skipLF初始化的时候直接赋值为false，所以这时候omitLF为false；nextChar和nChars在构造器中赋值为了0。  

由于是第一次执行readLine方法，所以1号if语句被执行，这时候**调用了fill()方法**，由于**readLine方法的前面没有对markedChar和readAheadLimit做过任何改动**，所以fill()方法真正被执行到的代码只有如下部分：  

	private void fill() throws IOException {
        int dst;
        if (markedChar <= UNMARKED) {
            /* No mark */
            dst = 0;
        } 
		.....
        int n;
        do {
            n = in.read(cb, dst, cb.length - dst);
        } while (n == 0);
        if (n > 0) {
            nChars = dst + n;
            nextChar = dst;
        }
    }  

重点关注**n = in.read(cb, dst, cb.length - dst);**，这里in这个Reader对象将新的数据读入到了cb这个缓冲数组中，且覆盖的范围是**0-cb.length**；且返回的是这一次写入到数组中的char的数量，也就是说加入数组长为20，但是这一次只填充了12个，那么返回的就是12.    

可见，**fill方法的作用是将cb数组填充好新的内容，将nextChar置为缓冲数组的第一个数据位置(0)，nChars置为数组的长度**。  

接着分析readLine方法，在执行完fill()方法后2号if语句就不会被执行了，又由于omitLF为false，所以3号if语句也不会被执行。  

重点是**charLoop循环**，这个循环除了遍历完所有的缓冲数组这种情况会被跳出外，当**遇到'\n','\r'**两个字符也会跳出循环。  

##### 1. 因为遍历完所有元素而退出循环  
紧接着，startChar被赋值为了本次遍历的起点，nextChar这时候已经和nChars相等了，都是数组的长度。同样5、6、7号if语句直接跳过，s这个StringBuffer将缓冲数组中所有的char都append了进去  

之后就是**下一次bufferLoop循环，这次循环一定会调用fill()方法将缓冲数组中的数据刷新**，后面的流程就是和之前一样了  

##### 2. 因为遇到'\n','\r'而退出循环  
当4号if语句被执行后，eol被赋值为true，然后跳出了循环。紧接着，startChar被赋值为了本次遍历的起点，**nextChar被赋值为了'\n'或者'\r'对应的数组下标**。  

由于eol为true，所以5号if语句被执行，6号if语句负责将本次遇到'\n'或者'\r'之前的数据，添加到str中(**需要注意的是，str中添加的数据不会包含'\n'和'\r'**)，紧接着**nextChar++**。如果，由于遇到'\r'而退出的charLoop循环，那么skipLF赋值为true，这里影响的是3号if语句，如果前一次最后遇到了'\r',本次的第一个是'\n'的话，直接nextChar++来跳过这个'\n'(**为了保证后面的i-startChar不为0**)。  

接着**return str；(不是进行下一次bufferLoop循环，而是直接return)**  

> **总结：1.返回null的情况：在fill中由于到达流的末尾，n为-1的情况：这时候1号if语句虽然被执行，但nextChar和nChars的值都不会被修改，仍是相等的状态；2号if语句被执行，如果之前的s中有内容就返回，否则返回null。  2. 通过上面的分析我们可以得到一些讯息：由于遇到'\n','\r'就会停止，而且紧接着nextCahr++也导致了下一次readLine也不会包括'\n','\r',所以数据流中的这两个符号都不会被记录住，造成字符的丢失；同样，当一个持续处于链接状态的数据流，一段数据发完后一定要以'\n'或者'\r'作为结尾，否则readLine不会返回，之前在写Socket和ServerSocket通信的时候由于客户端是通过PrintStream.print()写入的，导致server端的BufferedReader的readLine一直没法返回**  

