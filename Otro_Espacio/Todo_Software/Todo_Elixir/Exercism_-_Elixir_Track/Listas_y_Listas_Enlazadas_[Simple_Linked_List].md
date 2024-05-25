# 🙈Listas y Listas Enlazadas [Simple Linked List]
[Ejercicio](https://exercism.io/my/solutions/2dff8ebc803a419b9c6f7b8c00ff9b19).

Este me pareció sencillo a medida que avanzaba. Eso sí, usé funciones de la librería estándar. Así lo resolví:

    defmodule LinkedList do
      @opaque t :: tuple()
    
      @doc """
      Construct a new LinkedList
      """
      @spec new() :: t
      def new() do
        []
      end
    
      @doc """
      Push an item onto a LinkedList
      """
      @spec push(t, any()) :: t
      def push(list, elem) do
        [elem | list]
      end
    
      @doc """
      Calculate the length of a LinkedList
      """
      @spec length(t) :: non_neg_integer()
      def length(list) do
        Kernel.length(list)
      end
    
      @doc """
      Determine if a LinkedList is empty
      """
      @spec empty?(t) :: boolean()
      def empty?(list) do
        LinkedList.length(list) == 0
      end
    
      @doc """
      Get the value of a head of the LinkedList
      """
      @spec peek(t) :: {:ok, any()} | {:error, :empty_list}
      def peek([head | list]) do
        {:ok, head}
      end
    
      def peek([]) do
        {:error, :empty_list}
      end
    
      @doc """
      Get tail of a LinkedList
      """
      @spec tail(t) :: {:ok, t} | {:error, :empty_list}
      def tail([]) do
        {:error, :empty_list}
      end
    
      def tail([list | tail]) do
        {:ok, tail}
      end
    
      @doc """
      Remove the head from a LinkedList
      """
      @spec pop(t) :: {:ok, any(), t} | {:error, :empty_list}
      def pop([h | list]) do
        {:ok, h, list}
      end
    
      def pop([]) do
        {:error, :empty_list}
      end
    
      @doc """
      Construct a LinkedList from a stdlib List
      """
      @spec from_list(list()) :: t
      def from_list(list) do
        list
      end
    
      @doc """
      Construct a stdlib List LinkedList from a LinkedList
      """
      @spec to_list(t) :: list()
      def to_list(list) do
        list
      end
    
      @doc """
      Reverse a LinkedList
      """
      @spec reverse(t) :: t
      def reverse(list) do
        Enum.reverse(list)
      end
    end
    

En [un comentario](https://exercism.io/tracks/elixir/exercises/simple-linked-list/solutions/e29f9851685c4e50a21c98aaab313dad#solution-comment-27641) de un ejercicio donde también usan funciones de la librería estándar, mencionan que la gracia es hacer nuestro propio algoritmo. No le veo la gracia. No vine aquí a hacer algoritmos en un lenguaje que estoy aprendiendo.

